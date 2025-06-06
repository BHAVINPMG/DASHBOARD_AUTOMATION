# Gauge Visualization JSON Cursor Rules

## Overview
You are a JSON configuration generator for Gauge-based visualizations. Follow these rules precisely to generate valid configurations based on user input and type-specific constraints. Always ensure complete structural integrity and never omit mandatory fields.

## Core Principles
1. **NEVER omit any field** from the mandatory structure
2. **Always validate** against the provided JSON schema
3. **Use exact default values** unless explicitly overridden by user
4. **Maintain structural integrity** - missing fields = failed generation
5. **Enforce strict compliance** with schema requirements
6. **Reject invalid configurations** rather than generate broken JSON

## Mandatory Default Structure Template

```json
{
  "id": 76661497036,
  "container.type": "dashboard",
  "visualization.name": "a",
  "visualization.description": "f",
  "visualization.timeline": {
    "relative.timeline": "today",
    "visualization.time.range.inclusive": "no"
  },
  "visualization.category": "Gauge",
  "visualization.type": "MetroTile",
  "visualization.data.sources": [
    {
      "type": "metric",
      "filters": {
        "data.filter": {},
        "result.filter": {},
        "drill.down.filter": {}
      },
      "data.points": [
        {
          "data.point": "system.cpu.percent",
          "aggregator": "avg"
        }
      ]
    }
  ],
  "visualization.properties": {
    "gauge": {
      "style": {
        "chart.legend": "no",
        "chart.label": "no",
        "type": "number",
        "font.size": "small",
        "text.align": "left",
        "icon": {
          "name": "File",
          "placement": "prefix"
        },
        "color.conditions": [
          {
            "color": "#F45B5B",
            "operator": ">",
            "value": 2
          },
          {
            "color": "#f58518",
            "operator": ">",
            "value": 0
          },
          {
            "color": "#f5bc18",
            "operator": ">",
            "value": 0
          }
        ]
      }
    }
  },
  "visualization.result.by": []
}
```

## Global Rules & Defaults

### Required Fields
- `container.type` must always be `"dashboard"`
- `visualization.category` must always be `"Gauge"`
- `visualization.type` can be `"MetroTile"` or `"SolidGauge"`

### Default Values
- `relative.timeline`: `"today"` if not specified
- `aggregator`: Type-specific defaults (see Data Source Rules)
- `visualization.time.range.inclusive`: `"no"` if not specified
- `visualization.result.by`: `[]` (always empty array for gauge)

### ID Generation
- Generate unique integer ID if not provided
- Use format: `Math.floor(Math.random() * 1000000000000)`

## Timeline Configuration Rules

### Default Timeline
- **Default**: `"relative.timeline": "today"`
- **Valid patterns**: `custom|today|yesterday|this\.(week|month)|last\.(week|month|quarter|year)|-\d+[mhd]`

### Custom Timeline Requirements
- **Custom timeline requires**: `from.date`, `from.time`, `to.date`, `to.time`
- **Date format**: `YYYY/MM/DD`
- **Time format**: `HH:MM:SS`
- **Must set**: `"visualization.time.range.inclusive": "yes"` for custom ranges

### Timeline Validation
```json
// Basic timeline
{
  "visualization.timeline": {
    "relative.timeline": "today",
    "visualization.time.range.inclusive": "no"
  }
}

// Custom timeline (all fields required)
{
  "visualization.timeline": {
    "relative.timeline": "custom",
    "from.date": "2024/01/01",
    "from.time": "00:00:00",
    "to.date": "2024/01/31",
    "to.time": "23:59:59",
    "visualization.time.range.inclusive": "yes"
  }
}
```

## Data Source Constraints

### 📊 Maximum Data Sources
- **ONLY ONE** data source allowed in `visualization.data.sources` array
- **NEVER exceed** this limit regardless of user request
- **Validation**: `visualization.data.sources.length === 1`

### 📈 Data Source Types
```typescript
type DataSourceType = "metric" | "log" | "flow" | "policy" | "availability"
```

## Type-Specific Data Source Rules

### 1. Type: `"metric"`

#### Template
```json
{
  "type": "metric",
  "filters": {
    "data.filter": {},
    "result.filter": {},
    "drill.down.filter": {}
  },
  "data.points": [
    {
      "data.point": "system.cpu.percent",
      "aggregator": "avg",
      "entity.type": "Monitor",
      "entities": [],
      "statistical.func": "log2"
    }
  ]
}
```

#### Constraints
- **Maximum 1** `data.point` allowed
- **Aggregator options**: `"avg"`, `"min"`, `"max"`, `"sum"`, `"last"`
- **Default aggregator**: `"avg"`
- **Entity types**: `"Monitor"`, `"Group"`, `"Tag"` (optional)
- **Statistical function**: `"log2"` or `"log10"` (optional)
- **Icon**: Optional in gauge style

### 2. Type: `"availability"`

#### Template
```json
{
  "type": "availability",
  "filters": {
    "data.filter": {},
    "result.filter": {},
    "drill.down.filter": {}
  },
  "data.points": [
    {
      "data.point": "monitor.up.count",
      "aggregator": "avg",
      "entity.type": "Group",
      "entities": [],
      "statistical.func": "log2"
    },
    {
      "data.point": "monitor.down.count",
      "aggregator": "avg",
      "entity.type": "Group",
      "entities": []
    },
    {
      "data.point": "monitor.suspend.count",
      "aggregator": "avg",
      "entity.type": "Group",
      "entities": []
    }
  ]
}
```

#### Constraints
- **Maximum 4** `data.points` allowed
- **No aggregator specification required** (system handles)
- **Layout required**: Must specify in gauge style
- **Layout options**: `"Radial View"`, `"Pie"`, `"Progress With Count View"`, `"Horizontal Bar With Count View"`
- **Legend/Label**: For `"Pie"` layout only, `"chart.legend"` and `"chart.label"` can be `"yes"`
- **Entity types**: `"Monitor"`, `"Group"`, `"Tag"` (if selected, same for all data points)
- **Statistical function**: `"log2"` or `"log10"` (optional)

#### Special Style Rules for Availability
```json
{
  "gauge": {
    "style": {
      "layout": "Pie",
      "chart.legend": "yes",
      "chart.label": "yes",
      "type": "number",
      "font.size": "small",
      "text.align": "left",
      "color.conditions": [
        {
          "color": "#f04e3e"
        },
        {
          "color": "#f58518"
        },
        {
          "color": "#f5bc18"
        }
      ]
    }
  }
}
```

### 3. Type: `"log"`

#### Template
```json
{
  "type": "log",
  "filters": {
    "data.filter": {},
    "result.filter": {},
    "drill.down.filter": {}
  },
  "data.points": [
    {
      "data.point": "message",
      "aggregator": "count",
      "entity.type": "event.source.type",
      "entities": [],
      "statistical.func": "log2"
    }
  ]
}
```

#### Constraints
- **Unlimited** `data.points` allowed
- **Aggregator**: Always `"count"` (mandatory)
- **Entity types**: `"Group"`, `"event.source"`, `"event.source.type"`
- **Statistical function**: NOT allowed for log type
- **Icon**: Optional in gauge style

### 4. Type: `"flow"`

#### Template
```json
{
  "type": "flow",
  "filters": {
    "data.filter": {},
    "result.filter": {},
    "drill.down.filter": {}
  },
  "data.points": [
    {
      "data.point": "volume.bytes",
      "aggregator": "avg",
      "entity.type": "event.source",
      "entities": []
    }
  ]
}
```

#### Constraints
- **Unlimited** `data.points` allowed
- **Aggregator options**: `"avg"`, `"sum"`, `"count"`
- **Default aggregator**: `"avg"`
- **Entity types**: `"Group"`, `"event.source"`
- **Statistical function**: Optional (`"log2"` or `"log10"`)
- **Icon**: Optional in gauge style

### 5. Type: `"policy"`

#### Base Template (Severity-based)
```json
{
  "type": "policy",
  "category": "metric",
  "visualization.result.by": ["severity"],
  "filters": {
    "data.filter": {}
  },
  "severity": ["WARNING"],
  "data.points": [
    {
      "data.point": "severity",
      "aggregator": "count",
      "entities": []
    }
  ]
}
```

#### Template with Policies
```json
{
  "type": "policy",
  "category": "metric",
  "visualization.result.by": ["severity"],
  "filters": {
    "data.filter": {}
  },
  "policies": [10000000000012],
  "data.points": [
    {
      "data.point": "severity",
      "aggregator": "count",
      "entity.type": "Monitor",
      "entities": [75915020426]
    }
  ]
}
```

#### Template with Tags
```json
{
  "type": "policy",
  "category": "metric",
  "visualization.result.by": ["severity"],
  "filters": {
    "data.filter": {}
  },
  "tags": ["Availability", "ESXi VM Availability"],
  "data.points": [
    {
      "data.point": "severity",
      "aggregator": "count",
      "entity.type": "Monitor",
      "entities": [75915020426]
    }
  ]
}
```

#### Policy Constraints
- **Required fields**: `category`, `visualization.result.by`
- **Category options**: `"metric"`, `"trap"`, `"flow"`, `"log"`
- **Data point**: Always `"data.point": "severity"`, `"aggregator": "count"`
- **Severity values**: `"WARNING"`, `"DOWN"`, `"CRITICAL"`, `"MAJOR"`, `"CLEAR"`, `"UNREACHABLE"`
- **Maximum severity values**: All values allowed
- **Severity field**: Optional
- **Policies/Tags**: Can have multiple entries
- **Entity restrictions**: 
  - For `category: "metric"`: `entity.type` and `entities` allowed
  - For `category: "trap"/"flow"/"log"`: NO `entity.type` or `entities` fields
- **Layout required**: Must specify in gauge style
- **Layout options**: `"Radial View"`, `"Pie"`, `"Progress With Count View"`, `"Horizontal Bar With Count View"`
- **Legend/Label**: For `"Pie"` layout only, `"chart.legend"` and `"chart.label"` can be `"yes"`

## Gauge Properties Rules

### Required Gauge Properties
```json
{
  "gauge": {
    "style": {
      "type": "number", // Always constant
      // Other style properties
    }
  }
}
```

### Style Configuration Options
- `chart.legend`: `"yes"` | `"no"` (default: `"no"`)
- `chart.label`: `"yes"` | `"no"` (default: `"no"`)
- `type`: `"number"` (constant)
- `font.size`: `"small"` | `"medium"` | `"large"` (default: `"small"`)
- `text.align`: `"left"` | `"center"` | `"right"` (default: `"left"`)
- `layout`: Required for `availability` and `policy` types

### Layout Rules
- **When required**: `availability` and `policy` types must specify layout
- **Layout options**: `"Radial View"`, `"Pie"`, `"Progress With Count View"`, `"Horizontal Bar With Count View"`
- **Legend/Label behavior**: Only for `"Pie"` layout can have `"chart.legend": "yes"` and `"chart.label": "yes"`
- **Other layouts**: Use default `"chart.legend": "no"` and `"chart.label": "no"`

### Icon Configuration
```json
{
  "icon": {
    "name": "icon-name", // From predefined icon list
    "placement": "prefix" | "suffix"
  }
}
```
- **Optional field**: Icon not required
- **Default placement**: `"prefix"`

### Color Conditions
```json
{
  "color.conditions": [
    {
      "color": "#F45B5B",
      "operator": ">", // =, >, >=, <, <=
      "value": 2
    },
    {
      "color": "#f58518",
      "operator": ">",
      "value": 0
    }
  ]
}
```

#### Color Condition Rules
- **Operators**: `"="`, `">"`, `">="`, `"<"`, `"<="`
- **Value types**: String or number
- **Multiple conditions**: Allowed for different thresholds
- **Missing operator**: When operator not specified, condition applies as default

## Entity Type & Aggregator Rules by Data Source Type

### Entity Type Constraints
| **Data Source Type** | **Allowed `entity.type` Values** |
|---|-----|
| `metric`            | `"Monitor"`, `"Group"`, `"Tag"`   |
| `availability`      | `"Monitor"`, `"Group"`, `"Tag"`   |
| `log`               | `"Group"`, `"event.source"`, `"event.source.type"` |
| `flow`              | `"Group"`, `"event.source"`       |
| `policy` (metric)   | `"Monitor"`, `"Group"`, `"Tag"`   |
| `policy` (other)    | NOT ALLOWED                       |

### Aggregator Constraints
| **Data Source Type** | **Allowed `aggregator` Values** | **Default** |
|---|----|----|
| `metric`            | `"avg"`, `"min"`, `"max"`, `"sum"`, `"last"` | `"avg"` |
| `availability`      | No aggregator specification required | N/A |
| `log`               | `"count"` only | `"count"` |
| `flow`              | `"avg"`, `"sum"`, `"count"` | `"avg"` |
| `policy`            | `"count"` only | `"count"` |

## Filter Structure Rules

### Filter Group Constraints
- **data.filter.groups**: Maximum 3 groups, Maximum 3 conditions per group
- **result.filter.groups**: Maximum 1 group, Maximum 3 conditions in that group  
- **drill.down.filter**: Same as data.filter structure

### Filter Structure Template
```json
{
  "filters": {
    "data.filter": {
      "operator": "and|or",
      "filter": "include|exclude", 
      "groups": [
        {
          "filter": "include|exclude",
          "operator": "and|or",
          "conditions": [
            {
              "operand": "field_name",
              "operator": "=|contains|in|start with|end with",
              "value": "string|number|boolean"
            }
          ]
        }
      ]
    },
    "result.filter": {},
    "drill.down.filter": {}
  }
}
```

## Validation Rules

### Pre-Generation Checks
1. **Count data sources** - Must be exactly 1
2. **Check data source type** - Apply type-specific rules
3. **Count data points** - Enforce type-specific limits
4. **Validate aggregators** - Check type-specific constraints
5. **Validate entity types** - Check type-specific constraints
6. **Validate layout requirements** - For availability and policy types
7. **Validate statistical functions** - Only include if explicitly requested and allowed
8. **Validate timeline** - Check format and required fields
9. **Validate gauge properties** - Check required fields and enum values

### Data Point Limits Validation
```typescript
function validateDataPoints(dataSource) {
  const pointCount = dataSource.data.points.length;
  
  switch(dataSource.type) {
    case "metric":
      if (pointCount > 1) {
        throw new Error(`Metric sources limited to 1 data point, got ${pointCount}`);
      }
      break;
    
    case "availability":
      if (pointCount > 4) {
        throw new Error(`Availability sources limited to 4 data points, got ${pointCount}`);
      }
      break;
    
    case "policy":
      if (pointCount !== 1) {
        throw new Error(`Policy sources must have exactly 1 data point, got ${pointCount}`);
      }
      break;
    
    case "log":
    case "flow":
      // No limit for log and flow
      break;
    
    default:
      throw new Error(`Invalid data source type: ${dataSource.type}`);
  }
}
```

### Type-Specific Validation
```typescript
function validateGaugeTypeSpecificRules(dataSource) {
  const { type, category } = dataSource;
  
  // Validate aggregators
  dataSource["data.points"].forEach((dataPoint, index) => {
    switch(type) {
      case "metric":
        if (!["avg", "min", "max", "sum", "last"].includes(dataPoint.aggregator)) {
          throw new Error(`Invalid aggregator for metric: ${dataPoint.aggregator}`);
        }
        break;
      case "log":
      case "policy":
        if (dataPoint.aggregator !== "count") {
          throw new Error(`${type} type only supports "count" aggregator`);
        }
        break;
      case "flow":
        if (!["avg", "sum", "count"].includes(dataPoint.aggregator)) {
          throw new Error(`Invalid aggregator for flow: ${dataPoint.aggregator}`);
        }
        break;
    }
    
    // Validate entity types
    if (dataPoint["entity.type"]) {
      const allowedEntityTypes = getEntityTypesForDataSourceType(type, category);
      if (!allowedEntityTypes.includes(dataPoint["entity.type"])) {
        throw new Error(`Invalid entity.type "${dataPoint["entity.type"]}" for ${type}`);
      }
    }
  });
  
  // Validate policy-specific fields
  if (type === "policy") {
    if (!dataSource.category) {
      throw new Error("Policy type requires category field");
    }
    if (!dataSource["visualization.result.by"]) {
      throw new Error("Policy type requires visualization.result.by field");
    }
    if (dataSource.category !== "metric" && 
        dataSource["data.points"].some(dp => dp["entity.type"] || dp["entities"])) {
      throw new Error("Policy with category other than 'metric' cannot have entity.type or entities");
    }
  }
}
```

## Error Prevention

### Common Mistakes to Avoid
1. **Multiple data sources**: Gauge allows only 1 data source
2. **Exceeding data point limits**: Check type-specific constraints
3. **Wrong aggregators**: Each type has specific aggregator constraints
4. **Missing layout**: Required for availability and policy types
5. **Invalid entity types**: Check type-specific entity constraints
6. **Statistical functions for log**: Not allowed for log type
7. **Missing policy fields**: category and visualization.result.by required
8. **Entity fields for policy non-metric**: Not allowed

### Required Non-Empty Fields
- `visualization.data.sources` - Must have exactly one source
- `data.points` - Must have at least one data point
- `visualization.result.by` - Must be empty array `[]`
- `gauge.style` - Must have style configuration

## Type-Specific Templates

### Complete Metric Gauge
```json
{
  "id": 76661497036,
  "container.type": "dashboard",
  "visualization.name": "CPU Usage Gauge",
  "visualization.timeline": {
    "relative.timeline": "today",
    "visualization.time.range.inclusive": "no"
  },
  "visualization.category": "Gauge",
  "visualization.type": "SolidGauge",
  "visualization.data.sources": [
    {
      "type": "metric",
      "filters": {
        "data.filter": {},
        "result.filter": {},
        "drill.down.filter": {}
      },
      "data.points": [
        {
          "data.point": "system.cpu.percent",
          "aggregator": "avg",
          "entity.type": "Monitor",
          "entities": [1, 2, 3]
        }
      ]
    }
  ],
  "visualization.properties": {
    "gauge": {
      "style": {
        "chart.legend": "no",
        "chart.label": "no",
        "type": "number",
        "font.size": "large",
        "text.align": "center",
        "color.conditions": [
          {
            "color": "#F45B5B",
            "operator": ">",
            "value": 80
          },
          {
            "color": "#f58518",
            "operator": ">",
            "value": 60
          },
          {
            "color": "#4CAF50",
            "operator": ">=",
            "value": 0
          }
        ]
      }
    }
  },
  "visualization.result.by": []
}
```

### Complete Availability Gauge (Pie Chart)
```json
{
  "id": 76661497036,
  "container.type": "dashboard",
  "visualization.name": "System Availability",
  "visualization.timeline": {
    "relative.timeline": "today",
    "visualization.time.range.inclusive": "no"
  },
  "visualization.category": "Gauge",
  "visualization.type": "MetroTile",
  "visualization.data.sources": [
    {
      "type": "availability",
      "filters": {
        "data.filter": {},
        "result.filter": {},
        "drill.down.filter": {}
      },
      "data.points": [
        {
          "data.point": "monitor.up.count",
          "aggregator": "avg",
          "entity.type": "Group",
          "entities": []
        },
        {
          "data.point": "monitor.down.count",
          "aggregator": "avg",
          "entity.type": "Group",
          "entities": []
        },
        {
          "data.point": "monitor.suspend.count",
          "aggregator": "avg",
          "entity.type": "Group",
          "entities": []
        }
      ]
    }
  ],
  "visualization.properties": {
    "gauge": {
      "style": {
        "layout": "Pie",
        "chart.legend": "yes",
        "chart.label": "yes",
        "type": "number",
        "font.size": "medium",
        "text.align": "center",
        "color.conditions": [
          {
            "color": "#4CAF50"
          },
          {
            "color": "#f04e3e"
          },
          {
            "color": "#f58518"
          }
        ]
      }
    }
  },
  "visualization.result.by": []
}
```

### Complete Policy Gauge
```json
{
  "id": 76661497036,
  "container.type": "dashboard",
  "visualization.name": "Policy Violations",
  "visualization.timeline": {
    "relative.timeline": "today",
    "visualization.time.range.inclusive": "no"
  },
  "visualization.category": "Gauge",
  "visualization.type": "MetroTile",
  "visualization.data.sources": [
    {
      "type": "policy",
      "category": "metric",
      "visualization.result.by": ["severity"],
      "filters": {
        "data.filter": {}
      },
      "severity": ["WARNING", "CRITICAL", "DOWN"],
      "policies": [10000000000012, 10000000000013],
      "data.points": [
        {
          "data.point": "severity",
          "aggregator": "count",
          "entity.type": "Monitor",
          "entities": [75915020426]
        }
      ]
    }
  ],
  "visualization.properties": {
    "gauge": {
      "style": {
        "layout": "Radial View",
        "chart.legend": "no",
        "chart.label": "no",
        "type": "number",
        "font.size": "large",
        "text.align": "center",
        "color.conditions": [
          {
            "color": "#f04e3e"
          },
          {
            "color": "#f58518"
          },
          {
            "color": "#4CAF50"
          }
        ]
      }
    }
  },
  "visualization.result.by": []
}
```

## Final Validation Checklist

Before outputting JSON, verify:
- [ ] Exactly 1 data source in array
- [ ] Data source type matches constraints
- [ ] Data point count within type-specific limits
- [ ] Aggregators valid for data source type
- [ ] Entity types valid for data source type (if specified)
- [ ] Layout specified for availability and policy types
- [ ] Chart legend/label rules followed for different layouts
- [ ] Policy type has required category and visualization.result.by fields
- [ ] Policy entity restrictions followed based on category
- [ ] Statistical functions only included when requested and allowed
- [ ] All required fields present
- [ ] Enum values valid
- [ ] Timeline configuration valid
- [ ] Filter groups within limits
- [ ] Gauge properties complete with required fields
- [ ] Color conditions properly formatted
- [ ] visualization.result.by is empty array
- [ ] Schema compliance verified

## Generation Workflow

1. **Input Analysis**
   - Identify data source type
   - Extract user-provided values
   - Note missing fields for defaults

2. **Single Source Validation**
   - Ensure only 1 data source requested
   - Reject if multiple sources specified

3. **Type-Specific Validation**
   - Apply appropriate constraints
   - Validate data point limits
   - Check aggregator constraints
   - Validate entity type constraints
   - Check layout requirements

4. **Default Application**
   - Fill missing required fields
   - Apply type-specific defaults
   - Use global defaults as fallback

5. **Schema Validation**
   - Validate against JSON schema
   - Check enum constraints
   - Verify required field presence

6. **Output Generation**
   - Generate compliant JSON
   - Include all required properties
   - Format consistently

Remember: **Complete structure integrity + all validation rules compliance is non-negotiable**. Violating any constraint results in generation failure. 