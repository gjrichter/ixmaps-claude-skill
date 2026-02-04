---
name: create-ixmap
description: Creates interactive maps using ixMaps framework. Use when the user wants to create a map, visualize geographic data, or display data with bubble charts, choropleth maps, pie charts, or bar charts on a map.
argument-hint: "[filename] [options]"
allowed-tools: Write, Read, AskUserQuestion
---

# Create ixMap Skill

This skill helps you create interactive maps using the ixMaps framework quickly and easily.

Creates a complete HTML file with an interactive ixMaps visualization. You can specify the map type, data, and visualization style through simple parameters.

## Usage

```
/create-ixmap [options]
```

## Parameters

The skill accepts parameters in natural language. You can specify:

- **filename**: Output HTML filename (default: "map.html")
- **title**: Map title
- **data**: Data as JSON array or URL to data file
- **maptype**: Base map style (VT_TONER_LITE, OpenStreetMap, CartoDB - Positron, etc.)
- **center**: Map center coordinates {lat, lng}
- **zoom**: Initial zoom level
- **viztype**: Visualization type (BUBBLE, CHOROPLETH, PIE, BAR, DOT)
- **colorscheme**: Color array for visualization

## Examples

```bash
# Create a simple bubble map of Italian cities
/create-ixmap filename=citta_italia.html title="Città Italiane"

# Create a choropleth map
/create-ixmap viztype=CHOROPLETH data=regions.csv center={lat:42,lng:12} zoom=6

# Create with custom colors
/create-ixmap colorscheme=["#ff0000","#00ff00"] maptype=OpenStreetMap
```

## Instructions for Claude

When this skill is invoked, you should:

1. **Parse the parameters** from the user's request (either from command arguments or from conversational context)

2. **Gather required information** by asking the user if needed:
   - What data should be displayed? (cities, regions, points of interest)
   - What should the visualization show? (population, values, categories)
   - Any specific styling preferences?

3. **Generate the HTML file** with:
   - Complete HTML5 structure
   - ixMaps CDN script included
   - Responsive styling
   - Data embedded inline or loaded from URL
   - Proper ixMaps configuration using fluent API
   - Appropriate visualization type and styling

4. **Use the template structure**:
```html
<!DOCTYPE html>
<html lang="it">
<head>
    <meta charset="UTF-8">
    <title>[TITLE]</title>
    <script src="https://cdn.jsdelivr.net/gh/gjrichter/ixmaps_flat@master/ixmaps.js"></script>
    <style>
        body { margin: 0; padding: 0; }
        #map { width: 100%; height: 100vh; }
    </style>
</head>
<body>
    <div id="map"></div>
    <script>
        // Data
        const data = [DATA];

        // Create map
        ixmaps.Map("map", {
            mapType: "[MAPTYPE]"
        })
        .options({
            objectscaling: "dynamic",
            basemapopacity: 0.6
        })
        .view({
            center: { lat: [LAT], lng: [LNG] },
            zoom: [ZOOM]
        })
        .layer(
            ixmaps.layer("data_layer")
                .data({ obj: data, type: "json" })
                .binding({ geo: "lat|lon", value: "[VALUE_FIELD]", title: "[TITLE_FIELD]" })
                .style({
                    colorscheme: [COLORS],
                    normalsizevalue: [NORM_VALUE],
                    opacity: 0.7,
                    showdata: "true"
                })
                .meta({
                    tooltip: "{{theme.item.chart}}{{theme.item.data}}"
                })
                .type("[VIZ_TYPE]")
                .title("[LAYER_TITLE]")
                .define()
        );
    </script>
</body>
</html>
```

**IMPORTANT - Data Source Specific Rules:**

**For GeoJSON data sources:**
- Type MUST be `FEATURE` or `FEATURE|CHOROPLETH` (for colored regions)
- Binding MUST include: `{ geo: "geometry", value: "$item$" }`
- Style MUST include: `showdata: "true"`
- Meta MUST include: `{ tooltip: "{{theme.item.chart}}{{theme.item.data}}" }`
- Example for GeoJSON:
```javascript
.data({ url: "path/to/data.geojson", type: "geojson" })
.binding({ geo: "geometry", value: "$item$" })
.style({ colorscheme: ["#0066cc"], showdata: "true" })
.meta({ tooltip: "{{theme.item.chart}}{{theme.item.data}}" })
.type("FEATURE|CHOROPLETH")
```

**For GeoJSON Categorical Choropleth (String Fields)**

- Use when the value field is a category (e.g., country name, region code)
- Type MUST include `CATEGORICAL`: `FEATURE|CHOROPLETH|CATEGORICAL`
- Use a categorical colorscheme preset to auto-generate enough colors:
  - `colorscheme: ["1","tableau"]`
- Binding should use the categorical field as `value` and `title`:
  - `.binding({ geo: "geometry", value: "NAME_FIELD", title: "NAME_FIELD" })`



**For point data (CSV/JSON with lat/lon):**

- Use standard chart types: `CHART|BUBBLE|SIZE|VALUES`, `CHART|PIE`, etc.
- Binding uses: `{ geo: "lat|lon", value: "fieldname" }`
- Style MUST include: `showdata: "true"`
- Meta MUST include: `{ tooltip: "{{theme.item.chart}}{{theme.item.data}}" }`
- Example for points:
```javascript
.data({ obj: data, type: "json" })
.binding({ geo: "lat|lon", value: "population", title: "name" })
.style({ colorscheme: ["#0066cc"], showdata: "true" })
.meta({ tooltip: "{{theme.item.chart}}{{theme.item.data}}" })
.type("CHART|BUBBLE|SIZE|VALUES")
```

5. **Default values** if not specified:
   - filename: "ixmap.html"
   - maptype: "VT_TONER_LITE"
   - center: {lat: 42.5, lng: 12.5} (Italy)
   - zoom: 6
   - viztype: "CHART|BUBBLE|SIZE|VALUES"
   - colorscheme: ["#0066cc"]

5b. **Valid mapType values** (IMPORTANT - use exact names):
   - `"VT_TONER_LITE"` - Clean, minimal base map (default)
   - `"OpenStreetMap"` - Standard OSM
   - `"CartoDB - Positron"` - Light CartoDB style (note the spaces and dash)
   - `"CartoDB - Dark_Matter"` - Dark CartoDB style (note the spaces and dash)
   - `"Stamen Terrain"` - Terrain with hill shading
   - **CRITICAL**: CartoDB map types require spaces around the dash: `"CartoDB - Positron"` NOT `"CartoDB Positron"`

6. **Data handling**:
   - If user provides inline data as JSON array, embed it directly
   - If user provides a CSV/JSON file URL, use `.data({url: "...", type: "..."})`
   - If user describes data (e.g., "Italian cities"), create sample data
   - Ensure data has required fields: lat, lon, and at least one value field

7. **Binding Rules** (CRITICAL):
   - **ALWAYS** include `.binding()` method for every layer
   - For GeoJSON: `{ geo: "geometry", value: "$item$" }`
   - For point data: `{ geo: "lat|lon", value: "fieldname", title: "titlefield" }`
   - The `value` field is REQUIRED - without it the layer won't work
   - Use `"$item$"` as value for GeoJSON when displaying the geometry itself

7b. **Style Rules** (IMPORTANT):
   - **ALWAYS** include `showdata: "true"` in the `.style()` object
   - This enables data display on the map elements
   - Example: `.style({ colorscheme: ["#0066cc"], showdata: "true" })`

7c. **Meta Rules** (IMPORTANT):
   - **ALWAYS** include `.meta()` method after `.style()` and before `.type()`
   - Default tooltip template: `{ tooltip: "{{theme.item.chart}}{{theme.item.data}}" }`
   - Only customize if the user explicitly requests different tooltip content
   - Example: `.meta({ tooltip: "{{theme.item.chart}}{{theme.item.data}}" })`

8. **Methods that DO NOT exist** (NEVER use):
   - `.tooltip()` - Does not exist in ixMaps API
   - To show information on hover, use the `title` property in `.binding()` instead

9. **Validation**:
   - Check that data has geographical coordinates (lat/lon or geometry)
   - Verify color scheme is valid array
   - Ensure visualization type matches data structure
   - VERIFY `.binding()` is always present with required properties

10. **After creation**:
   - Confirm the file was created
   - Explain what the map shows
   - Suggest how to open it (in browser)
   - Offer to modify or enhance the map

## Supported Visualization Types

### For Point Data (CSV/JSON with lat/lon coordinates):
- **CHART|BUBBLE|SIZE|VALUES** - Proportional circles
- **CHART|PIE** - Pie charts at locations
- **CHART|BAR|VALUES** - Bar charts
- **DOT** - Simple dots at locations

### For GeoJSON/Geometry Data:
- **FEATURE** - Simple geographic features (polygons, lines)
- **FEATURE|CHOROPLETH** - Color-coded regions with data values
  - Add `|EQUIDISTANT` for equal intervals: `FEATURE|CHOROPLETH|EQUIDISTANT`
  - Add `|QUANTILE` for quantiles: `FEATURE|CHOROPLETH|QUANTILE`

## Complete Examples

### Example 1: Point data with bubbles (CSV/JSON)
```javascript
ixmaps.layer("cities")
    .data({ obj: cityData, type: "json" })
    .binding({
        geo: "lat|lon",
        value: "population",
        title: "name"
    })
    .style({
        colorscheme: ["#0066cc"],
        showdata: "true"  // REQUIRED
    })
    .meta({
        tooltip: "{{theme.item.chart}}{{theme.item.data}}"  // REQUIRED
    })
    .type("CHART|BUBBLE|SIZE|VALUES")
    .title("Italian Cities by Population")
    .define()
```

### Example 2: GeoJSON with choropleth coloring
```javascript
ixmaps.layer("regions")
    .data({ url: "regions.geojson", type: "geojson" })
    .binding({
        geo: "geometry",
        value: "$item$"  // REQUIRED for GeoJSON
    })
    .style({
        colorscheme: ["#ffffcc","#ff0000"],
        opacity: 0.7,
        showdata: "true"  // REQUIRED
    })
    .meta({
        tooltip: "{{theme.item.chart}}{{theme.item.data}}"  // REQUIRED
    })
    .type("FEATURE|CHOROPLETH|EQUIDISTANT")
    .title("Regions")
    .define()
```

### Example 3: Simple GeoJSON features (no data values)
```javascript
ixmaps.layer("boundaries")
    .data({ url: "boundaries.geojson", type: "geojson" })
    .binding({
        geo: "geometry",
        value: "$item$"  // Still REQUIRED even without data
    })
    .style({
        fillcolor: "#cccccc",
        fillopacity: 0.3,
        showdata: "true"  // REQUIRED
    })
    .meta({
        tooltip: "{{theme.item.chart}}{{theme.item.data}}"  // REQUIRED
    })
    .type("FEATURE")
    .title("Administrative Boundaries")
    .define()
```

### Example 4: GeoJSON Categorical Choropleth (String Fields)

```javascript
ixmaps.layer("regions")
    .data({ url: "regions.topojson", type: "topojson" })
    .binding({ geo: "geometry", value: "NAME_ENGL", title: "NAME_ENGL" })
    .style({
        colorscheme: ["1","tableau"],
        showdata: "true"
    })
    .meta({ tooltip: "{{theme.item.chart}}{{theme.item.data}}" })
    .type("FEATURE|CHOROPLETH|CATEGORICAL")
    .title("Regions by Name")
    .define()
```

## 



## Notes

- The skill always creates a standalone HTML file that works without a server
- All dependencies are loaded from CDN
- Maps are fully interactive with zoom, pan, and hover information
- Data can be inline (for small datasets) or external URL (for large datasets)
- **CRITICAL**: Always use `.binding()` with appropriate `geo` and `value` properties
- **CRITICAL**: Always include `showdata: "true"` in `.style()`
- **CRITICAL**: Always include `.meta({ tooltip: "{{theme.item.chart}}{{theme.item.data}}" })`
- **NEVER** use `.tooltip()` - it doesn't exist in the ixMaps API

## Layer Method Chain Order

The correct order for layer methods is:
1. `.data()` - Define data source
2. `.binding()` - Map data fields to map properties
3. `.style()` - Visual styling (MUST include `showdata: "true"`)
4. `.meta()` - Metadata and tooltip configuration
5. `.type()` - Visualization type
6. `.title()` - Layer title
7. `.define()` - Finalize layer definition
