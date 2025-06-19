# OOTB Dashboard JSON Automation

A comprehensive collection of Out-of-the-Box (OOTB) dashboard JSON configurations and visualization rules for automated dashboard generation across multiple monitoring platforms.

## 🎯 Project Overview

This repository contains a complete set of JSON configurations, visualization rules, and examples for creating automated dashboards. It supports multiple visualization types including Charts, Grids, Gauges, and TopN visualizations across various data sources like metrics, logs, flows, and policies.

## 📁 Project Structure

```
OOTB DASHBOARD JSON/
├── .cursor/rules/                    # Cursor rules for visualization generation
│   ├── chart_visualization_rules.mdc
│   ├── grid_visualization_rules.mdc
│   ├── gauge_visualization_rules.mdc
│   ├── topn_visualization_rules.mdc
│   ├── kpi_selection_rules.mdc
│   └── kpi_selection_counters_*.md   # Categorized counter mappings
├── CHART EXAMPLES/                   # Chart visualization examples
│   ├── chart_schema.json
│   ├── cpu_utilization_*.json
│   ├── memory_utilization_*.json
│   └── ...
├── GRID EXAMPLES/                    # Grid visualization examples
│   ├── grid_schema.json
│   ├── cpu_utilization_grid.json
│   ├── availability_grid.json
│   └── ...
├── GAUGE EXAMPLES/                   # Gauge visualization examples
│   ├── gauge_schema.json
│   ├── cpu_utilization_*.json
│   └── ...
├── TopN Examples/                    # TopN visualization examples
│   ├── topn_schema.json
│   └── ...
├── EXISTING DASHBOARDS JSONS/        # Original dashboard configurations
│   ├── aws cloud.json
│   ├── azure cloud.json
│   ├── vmware.json
│   └── ...
└── README.md                         # This file
```

## 🚀 Features

### 📊 Supported Visualization Types

1. **Charts**
   - Area Chart
   - Line Chart
   - Horizontal Bar Chart
   - Vertical Bar Chart
   - Stacked Area Chart
   - Stacked Line Chart
   - Stacked Horizontal Bar Chart
   - Stacked Vertical Bar Chart

2. **Grids**
   - Standard Grid Layout
   - Key-Value Layout
   - Sortable columns
   - Color-coded thresholds
   - Searchable content

3. **Gauges**
   - MetroTile Gauge
   - Solid Gauge
   - Radial Gauge
   - Pie Gauge
   - Progress indicators

4. **TopN Visualizations**
   - TopN Area Chart
   - TopN Line Chart
   - TopN Bar Charts
   - TopN Pie Chart
   - TopN Grid
   - TopN Solid Gauge View

### 🔧 Data Source Support

- **Metrics**: System counters, performance metrics
- **Logs**: Event logs, message counts
- **Flows**: Network traffic, volume data
- **Policies**: Security violations, compliance data
- **Availability**: Uptime, monitoring status

### 🎨 Visualization Features

- **Color-coded thresholds** for performance indicators
- **Responsive layouts** for different screen sizes
- **Interactive elements** with drill-down capabilities
- **Real-time data** support with configurable time ranges
- **Customizable aggregators** (avg, min, max, sum, count)
- **Entity filtering** by monitors, groups, and tags

## 📋 Prerequisites

- Git repository access
- JSON-compatible dashboard platform
- Understanding of monitoring metrics and data sources

## 🛠️ Installation & Setup

1. **Clone the repository**
   ```bash
   git clone https://github.com/BHAVINPMG/DASHBOARD_AUTOMATION.git
   cd DASHBOARD_AUTOMATION
   ```

2. **Explore the structure**
   - Review the `.cursor/rules/` directory for visualization rules
   - Check example folders for JSON configurations
   - Examine existing dashboards in `EXISTING DASHBOARDS JSONS/`

3. **Understand the schemas**
   - Each visualization type has its own schema file
   - Schemas define the required structure and constraints

## 📖 Usage Guide

### Creating New Visualizations

1. **Choose your visualization type** (Chart, Grid, Gauge, TopN)
2. **Select appropriate example** from the corresponding folder
3. **Modify the JSON configuration** based on your requirements
4. **Validate against the schema** to ensure compliance
5. **Deploy to your dashboard platform**

### Using Cursor Rules

The `.cursor/rules/` directory contains comprehensive rules for:
- **Data source validation**
- **Visualization type constraints**
- **ID generation standards**
- **Color coding rules**
- **Aggregator compatibility**

### Example: Creating a CPU Utilization Chart

```json
{
  "id": 76335282565,
  "container.type": "dashboard",
  "visualization.name": "CPU Utilization Chart",
  "visualization.category": "Chart",
  "visualization.type": "Line",
  "visualization.data.sources": [
    {
      "type": "metric",
      "data.points": [
        {
          "data.point": "system.cpu.percent",
          "aggregator": "avg"
        }
      ]
    }
  ],
  "visualization.result.by": ["monitor"]
}
```

## 🔍 Key Components

### System Counter Mapping

The project includes automatic mapping from generic terms to system counters:
- **CPU** → `system.cpu.percent`
- **Memory** → `system.memory.usage.percent`
- **Disk** → `system.disk.used.percent`
- **Network** → `system.network.in.bytes`, `system.network.out.bytes`

### KPI Selection Rules

Comprehensive KPI mappings organized by service categories:
- **Application Services**
- **Cloud Services**
- **Database Services**
- **Network Security**
- **Server OS**
- **Storage Infrastructure**

### Visualization Rules

Strict validation rules ensure:
- **Data source compatibility**
- **Aggregator constraints**
- **Entity type validation**
- **Result grouping rules**
- **Color condition formatting**

## 🎯 Use Cases

### 1. **Infrastructure Monitoring**
- CPU, memory, and disk utilization charts
- Network traffic analysis
- Service availability tracking

### 2. **Cloud Platform Monitoring**
- AWS, Azure, Google Cloud metrics
- Container and virtualization monitoring
- Auto-scaling performance analysis

### 3. **Security & Compliance**
- Policy violation tracking
- Security event analysis
- Compliance dashboard creation

### 4. **Application Performance**
- Database performance monitoring
- Web server metrics
- Application service health

## 🔧 Customization

### Adding New Metrics

1. **Update KPI selection rules** in `.cursor/rules/`
2. **Add counter mappings** to appropriate category files
3. **Create example visualizations** for new metrics
4. **Update system counter mapping** if applicable

### Creating Custom Visualizations

1. **Follow the schema structure** for your visualization type
2. **Use appropriate data source types** and aggregators
3. **Apply color coding rules** for thresholds
4. **Validate against rules** before deployment

## 📊 Performance Considerations

- **Data point limits** are enforced per visualization type
- **Aggregator selection** affects performance and accuracy
- **Time range selection** impacts data processing
- **Entity filtering** helps reduce data volume

## 🤝 Contributing

1. **Fork the repository**
2. **Create a feature branch**
3. **Follow the existing structure** and naming conventions
4. **Validate your changes** against schemas and rules
5. **Submit a pull request** with detailed description

## 📝 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🆘 Support

For issues, questions, or contributions:
- **Create an issue** in the GitHub repository
- **Review existing examples** for guidance
- **Check the rules documentation** for validation requirements

## 🔄 Version History

- **v1.0.0**: Initial release with comprehensive visualization support
- **v1.1.0**: Added system counter mapping and KPI selection rules
- **v1.2.0**: Enhanced validation rules and example organization
- **v1.3.0**: Final cleanup and deployment readiness

---

**Ready for Production Use** 🚀

This project provides a complete foundation for automated dashboard generation with comprehensive validation rules, extensive examples, and production-ready configurations. 