# ixMaps Claude Skill

A Claude Code skill for creating interactive maps using the [ixMaps framework](https://github.com/gjrichter/ixmaps).

![ixMaps](https://img.shields.io/badge/ixMaps-Interactive%20Maps-blue)
![Claude Code](https://img.shields.io/badge/Claude-Code%20Skill-purple)
![License](https://img.shields.io/badge/license-MIT-green)

## 🗺️ What is this?

This skill enables Claude Code to generate interactive HTML maps with ixMaps using simple natural language commands. Perfect for data visualization, geographic analysis, and creating beautiful interactive maps.

## ✨ Features

- **Multiple visualization types**: Bubble charts, choropleth maps, pie charts, bar charts
- **GeoJSON support**: Full support for GeoJSON features with proper binding
- **Interactive tooltips**: Automatic tooltip generation with customization options
- **Multiple base maps**: VT_TONER_LITE, OpenStreetMap, CartoDB Positron/Dark_Matter, Stamen Terrain
- **Flexible data input**: Inline JSON, CSV files, GeoJSON, external URLs
- **Standalone output**: No server needed, works offline with CDN resources

## 🚀 Quick Start

### Installation

1. Clone this repository:
```bash
git clone https://github.com/gjrichter/ixmaps-claude-skill.git
```

2. Copy the skill to your Claude Code skills directory:
```bash
mkdir -p ~/.claude/skills/create-ixmap
cp ixmaps-claude-skill/SKILL.md ~/.claude/skills/create-ixmap/
cp ixmaps-claude-skill/template.html ~/.claude/skills/create-ixmap/
```

3. Restart Claude Code or reload skills

### Usage

In Claude Code, simply invoke the skill:

```bash
/create-ixmap
```

Or with parameters:

```bash
/create-ixmap filename=my_map.html title="My Custom Map"
```

## 📖 Examples

### Example 1: Point Data with Bubbles

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
        showdata: "true"
    })
    .meta({
        tooltip: "{{theme.item.chart}}{{theme.item.data}}"
    })
    .type("CHART|BUBBLE|SIZE|VALUES")
    .title("Italian Cities by Population")
    .define()
```

### Example 2: GeoJSON Choropleth Map

```javascript
ixmaps.layer("regions")
    .data({ url: "regions.geojson", type: "geojson" })
    .binding({
        geo: "geometry",
        value: "$item$"
    })
    .style({
        colorscheme: ["#ffffcc","#ff0000"],
        opacity: 0.7,
        showdata: "true"
    })
    .meta({
        tooltip: "{{theme.item.chart}}{{theme.item.data}}"
    })
    .type("FEATURE|CHOROPLETH|EQUIDISTANT")
    .title("Regions")
    .define()
```

See the [examples](./examples/) directory for complete working examples.

## 📋 Supported Visualization Types

### For Point Data (lat/lon)
- `CHART|BUBBLE|SIZE|VALUES` - Proportional circles
- `CHART|PIE` - Pie charts at locations
- `CHART|BAR|VALUES` - Bar charts
- `DOT` - Simple dots

### For GeoJSON Data
- `FEATURE` - Simple geographic features
- `FEATURE|CHOROPLETH` - Color-coded regions
- `FEATURE|CHOROPLETH|EQUIDISTANT` - Equal interval coloring
- `FEATURE|CHOROPLETH|QUANTILE` - Quantile-based coloring

## 🗺️ Supported Map Types

- `VT_TONER_LITE` - Clean, minimal base map (default)
- `OpenStreetMap` - Standard OSM
- `CartoDB - Positron` - Light CartoDB style
- `CartoDB - Dark_Matter` - Dark CartoDB style
- `Stamen Terrain` - Terrain with hill shading

## ⚙️ Critical Rules

The skill follows ixMaps best practices:

### For All Layers (REQUIRED)
- ✅ `.binding()` with appropriate `geo` and `value` properties
- ✅ `showdata: "true"` in `.style()` object
- ✅ `.meta({ tooltip: "{{theme.item.chart}}{{theme.item.data}}" })` method

### For GeoJSON Specifically
- ✅ Type MUST be `FEATURE` or `FEATURE|CHOROPLETH`
- ✅ Binding MUST include: `{ geo: "geometry", value: "$item$" }`
- ❌ NEVER use regular chart types (BUBBLE, PIE, BAR) with GeoJSON
- ❌ NEVER use `.tooltip()` method (it doesn't exist in ixMaps)

### Layer Method Chain Order
1. `.data()` - Define data source
2. `.binding()` - Map data fields to map properties
3. `.style()` - Visual styling (MUST include `showdata: "true"`)
4. `.meta()` - Metadata and tooltip configuration
5. `.type()` - Visualization type
6. `.title()` - Layer title
7. `.define()` - Finalize layer definition

## 📚 Documentation

- [SKILL.md](./SKILL.md) - Complete skill documentation for Claude
- [template.html](./template.html) - HTML template with placeholders
- [examples/](./examples/) - Working HTML examples

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- [ixMaps](https://github.com/gjrichter/ixmaps_flat) - The interactive mapping framework
- [Claude Code](https://claude.com/claude-code) - AI-powered coding assistant

## 📧 Contact

For questions or feedback about this skill, please open an issue on GitHub.

## 🔗 Related Projects

- [ixMaps Framework](https://github.com/gjrichter/ixmaps_flat)
- [ixMaps Documentation](https://gjrichter.github.io/ixmaps_flat/)
- [Claude Code Skills](https://github.com/topics/claude-code-skill)

---

Made with ❤️ for the ixMaps and Claude Code communities
