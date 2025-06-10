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
        "text.align": "left"
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
- **`font.size`: `"small"` if not specified (MANDATORY DEFAULT)**
- **`text.align`: `"left"` if not specified (MANDATORY DEFAULT)**
- **No threshold values by default - only add color.conditions when explicitly requested**

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
      "aggregator": "avg"
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
- **Default style**: `font.size: "small"`, `text.align: "left"`, no color conditions

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
```

#### Constraints
- **Maximum 4** `data.points` allowed
- **No aggregator specification required** (system handles)
- **Layout required**: Must specify in gauge style
- **Layout options**: `"Radial View"`, `"Pie"`, `"Progress With Count View"`, `"Horizontal Bar With Count View"`
- **Legend/Label**: For `"Pie"` layout only, `"chart.legend"` and `"chart.label"` can be `"yes"`
- **Entity types**: `"Monitor"`, `"Group"`, `"Tag"` (if selected, same for all data points)
- **Statistical function**: `"log2"` or `"log10"` (optional)
- **Default style**: `font.size: "small"`, `text.align: "left"`, no color conditions

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
      "aggregator": "count"
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
- **Default style**: `font.size: "small"`, `text.align: "left"`, no color conditions

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
      "aggregator": "avg"
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
- **MANDATORY FIELDS**: `font.size` and `text.align` must ALWAYS be present with defaults if not specified
- **Default style**: `font.size: "small"`, `text.align: "left"`, no color conditions

#### **Special Flow Style Requirements**
```json
{
  "gauge": {
    "style": {
      "chart.legend": "no",
      "chart.label": "no", 
      "type": "number",
      "font.size": "small",  // ALWAYS REQUIRED - use default if not specified
      "text.align": "left"   // ALWAYS REQUIRED - use default if not specified
    }
  }
}
```

### 5. Type: `"policy"`

#### Base Template (Default - No Severity, No Entity)
```json
{
  "type": "policy",
  "category": "metric",
  "visualization.result.by": ["severity"],
  "filters": {
    "data.filter": {}
  },
  "data.points": [
    {
      "data.point": "severity",
      "aggregator": "count"
    }
  ]
}
```

#### Policy Constraints
- **Required fields**: `category`, `visualization.result.by`
- **Category options**: `"metric"`, `"trap"`, `"flow"`, `"log"`
- **Data point**: Always `"data.point": "severity"`, `"aggregator": "count"`
- **DEFAULT LAYOUT**: `"Pie"` (mandatory default)
- **DEFAULT: NO severity field** - only add if explicitly requested by user
- **DEFAULT: NO entity.type or entities fields** - only add if explicitly requested by user
- **Severity values**: `"WARNING"`, `"DOWN"`, `"CRITICAL"`, `"MAJOR"`, `"CLEAR"`, `"UNREACHABLE"`
- **Maximum severity values**: All values allowed
- **Policies/Tags**: Can have multiple entries (only if user specifies)
- **Entity restrictions**: 
  - For `category: "metric"`: `entity.type` and `entities` allowed ONLY if user explicitly requests
  - For `category: "trap"/"flow"/"log"`: NO `entity.type` or `entities` fields
- **Layout required**: Must specify `"Pie"` by default
- **Default style**: `font.size: "small"`, `text.align: "left"`, no color conditions

#### **Policy Default Template**
```json
{
  "type": "policy",
  "category": "metric",
  "visualization.result.by": ["severity"],
  "filters": {
    "data.filter": {}
  },
  "data.points": [
    {
      "data.point": "severity",
      "aggregator": "count"
      // NO entity.type by default
      // NO entities by default
    }
  ]
  // NO severity field by default
}
```

#### **Policy Style Template with Pie Default**
```json
{
  "gauge": {
    "style": {
      "layout": "Pie",           // MANDATORY DEFAULT
      "chart.legend": "no",      // Default for Pie can be "yes" if user requests
      "chart.label": "no",       // Default for Pie can be "yes" if user requests
      "type": "number",
      "font.size": "small",
      "text.align": "left"
    }
  }
}
```

## Gauge Properties Rules

### Required Gauge Properties
```json
{
  "gauge": {
    "style": {
      "type": "number", // Always constant
      "font.size": "small", // MANDATORY DEFAULT
      "text.align": "left"  // MANDATORY DEFAULT
      // Other style properties
    }
  }
}
```

### Style Configuration Options
- `chart.legend`: `"yes"` | `"no"` (default: `"no"`)
- `chart.label`: `"yes"` | `"no"` (default: `"no"`)
- `type`: `"number"` (constant)
- **`font.size`: `"small"` | `"medium"` | `"large"` (MANDATORY DEFAULT: `"small"`)**
- **`text.align`: `"left"` | `"center"` | `"right"` (MANDATORY DEFAULT: `"left"`)**
- `layout`: Required for `availability` and `policy` types

### Layout Rules
- **When required**: `availability` and `policy` types must specify layout
- **Layout options**: `"Radial View"`, `"Pie"`, `"Progress With Count View"`, `"Horizontal Bar With Count View"`
- **Policy default layout**: `"Pie"` (mandatory default)
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
- **DEFAULT BEHAVIOR**: NO color.conditions field by default
- **Only add when user explicitly requests thresholds or color coding**
- **When added**:
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
- **DEFAULT**: Do not include color.conditions unless user explicitly requests thresholds

## Updated Default Application Rules

### Default Style Application
```typescript
function applyDefaultGaugeStyle(dataSource, userStyle) {
  const { type } = dataSource;
  
  // Mandatory defaults for ALL types
  const defaultStyle = {
    "chart.legend": userStyle["chart.legend"] || "no",
    "chart.label": userStyle["chart.label"] || "no", 
    "type": "number",
    "font.size": userStyle["font.size"] || "small",  // MANDATORY
    "text.align": userStyle["text.align"] || "left"  // MANDATORY
  };
  
  // Special handling for flow type - ALWAYS include font.size and text.align
  if (type === "flow") {
    defaultStyle["font.size"] = userStyle["font.size"] || "small";
    defaultStyle["text.align"] = userStyle["text.align"] || "left";
  }
  
  // Policy type gets Pie layout by default
  if (type === "policy") {
    defaultStyle["layout"] = userStyle["layout"] || "Pie";
  }
  
  // Availability type requires layout (user must specify)
  if (type === "availability" && !userStyle["layout"]) {
    throw new Error("Availability type requires layout specification");
  }
  
  // Only add color.conditions if user explicitly requests
  if (userStyle["color.conditions"]) {
    defaultStyle["color.conditions"] = userStyle["color.conditions"];
  }
  
  // Only add icon if user specifies
  if (userStyle["icon"]) {
    defaultStyle["icon"] = userStyle["icon"];
  }
  
  return defaultStyle;
}
```

### Policy Default Application
```typescript
function applyPolicyDefaults(dataSource, userInput) {
  const policyDefaults = {
    "category": dataSource.category || "metric",
    "visualization.result.by": dataSource["visualization.result.by"] || ["severity"],
    "filters": {
      "data.filter": {}
    },
    "data.points": [
      {
        "data.point": "severity",
        "aggregator": "count"
        // NO entity.type by default
        // NO entities by default
      }
    ]
    // NO severity field by default
  };
  
  // Only add severity if user explicitly requests it
  if (userInput.includes("severity") && userInput.includes("WARNING|CRITICAL|DOWN|MAJOR|CLEAR|UNREACHABLE")) {
    policyDefaults["severity"] = extractSeverityFromInput(userInput);
  }
  
  // Only add entity.type if user explicitly requests it
  if (userInput.includes("entity") || userInput.includes("monitor") || userInput.includes("group")) {
    policyDefaults["data.points"][0]["entity.type"] = extractEntityTypeFromInput(userInput);
    policyDefaults["data.points"][0]["entities"] = [];
  }
  
  return policyDefaults;
}
```

### Flow Type Validation
```typescript
function validateFlowTypeStyle(styleConfig) {
  // Flow type MUST have font.size and text.align
  if (!styleConfig["font.size"]) {
    styleConfig["font.size"] = "small";
  }
  
  if (!styleConfig["text.align"]) {
    styleConfig["text.align"] = "left";
  }
  
  return styleConfig;
}
```

## Entity Type & Aggregator Rules by Data Source Type

### Entity Type Constraints
| **Data Source Type** | **Allowed `entity.type` Values** | **Default Behavior** |
|---|-----|-----|
| `metric`            | `"Monitor"`, `"Group"`, `"Tag"`   | Optional - only add if user requests |
| `availability`      | `"Monitor"`, `"Group"`, `"Tag"`   | Optional - only add if user requests |
| `log`               | `"Group"`, `"event.source"`, `"event.source.type"` | Optional - only add if user requests |
| `flow`              | `"Group"`, `"event.source"`       | Optional - only add if user requests |
| `policy` (metric)   | `"Monitor"`, `"Group"`, `"Tag"`   | **DEFAULT: NOT INCLUDED** - only add if user explicitly requests |
| `policy` (other)    | NOT ALLOWED                       | **DEFAULT: NOT INCLUDED** |

### Aggregator Constraints
| **Data Source Type** | **Allowed `aggregator` Values** | **Default** |
|---|----|----|
| `metric`            | `"avg"`, `"min"`, `"max"`, `"sum"`, `"last"` | `"avg"` |
| `availability`      | No aggregator specification required | N/A |
| `log`               | `"count"` only | `"count"` |
| `flow`              | `"avg"`, `"sum"`, `"count"` | `"avg"` |
| `policy`            | `"count"` only | `"count"` |

## Updated Validation Rules

### Pre-Generation Checks
1. **Count data sources** - Must be exactly 1
2. **Check data source type** - Apply type-specific rules
3. **Count data points** - Enforce type-specific limits
4. **Validate aggregators** - Check type-specific constraints
5. **Validate entity types** - Check type-specific constraints (only if user requests)
6. **Validate layout requirements** - For availability and policy types
7. **Validate statistical functions** - Only include if explicitly requested and allowed
8. **Validate timeline** - Check format and required fields
9. **Validate gauge properties** - Check required fields and enum values
10. **Apply mandatory defaults** - font.size, text.align, policy layout
11. **Validate flow requirements** - Ensure font.size and text.align always present

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
    
    // Validate entity types (only if present)
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

### Style Validation
```typescript
function validateGaugeStyle(styleConfig, dataSourceType) {
  // Ensure mandatory defaults
  if (!styleConfig["font.size"]) {
    styleConfig["font.size"] = "small";
  }
  
  if (!styleConfig["text.align"]) {
    styleConfig["text.align"] = "left";
  }
  
  // Special validation for flow type
  if (dataSourceType === "flow") {
    if (!styleConfig["font.size"] || !styleConfig["text.align"]) {
      throw new Error("Flow type must have font.size and text.align fields");
    }
  }
  
  // Policy gets Pie layout by default
  if (dataSourceType === "policy" && !styleConfig["layout"]) {
    styleConfig["layout"] = "Pie";
  }
  
  return styleConfig;
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
9. **Missing mandatory style defaults**: font.size and text.align must be present
10. **Adding color.conditions by default**: Only add when user explicitly requests
11. **Adding severity/entity for policy by default**: Only add when user explicitly requests
12. **Missing flow style requirements**: font.size and text.align must always be present for flow

### Required Non-Empty Fields
- `visualization.data.sources` - Must have exactly one source
- `data.points` - Must have at least one data point
- `visualization.result.by` - Must be empty array `[]`
- `gauge.style` - Must have style configuration with mandatory defaults
- **`gauge.style.font.size`** - Must be present (default: "small")
- **`gauge.style.text.align`** - Must be present (default: "left")

## Type-Specific Templates

### Complete Metric Gauge (Minimal Default)
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
        "text.align": "left"
      }
    }
  },
  "visualization.result.by": []
}
```

### Complete Flow Gauge (Mandatory Style Fields)
```json
{
  "id": 76661497036,
  "container.type": "dashboard",
  "visualization.name": "Volume Bytes Flow Gauge",
  "visualization.timeline": {
    "relative.timeline": "today",
    "visualization.time.range.inclusive": "no"
  },
  "visualization.category": "Gauge",
  "visualization.type": "SolidGauge",
  "visualization.data.sources": [
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
        "font.size": "small",     // MANDATORY for flow
        "text.align": "left"     // MANDATORY for flow
      }
    }
  },
  "visualization.result.by": []
}
```

### Complete Policy Gauge (Default Pie, No Severity/Entity)
```json
{
  "id": 76661497036,
  "container.type": "dashboard",
  "visualization.name": "Policy Violations Gauge",
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
      "data.points": [
        {
          "data.point": "severity",
          "aggregator": "count"
        }
      ]
    }
  ],
  "visualization.properties": {
    "gauge": {
      "style": {
        "layout": "Pie",          // DEFAULT for policy
        "chart.legend": "no",
        "chart.label": "no",
        "type": "number",
        "font.size": "small",
        "text.align": "left"
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
- [ ] Layout specified for availability and policy types (Pie default for policy)
- [ ] Chart legend/label rules followed for different layouts
- [ ] Policy type has required category and visualization.result.by fields
- [ ] Policy entity restrictions followed based on category
- [ ] **Policy defaults applied: no severity field, no entity.type, Pie layout**
- [ ] Statistical functions only included when requested and allowed
- [ ] All required fields present
- [ ] Enum values valid
- [ ] Timeline configuration valid
- [ ] Filter groups within limits
- [ ] Gauge properties complete with required fields
- [ ] **Mandatory style defaults applied: font.size="small", text.align="left"**
- [ ] **Flow type has mandatory font.size and text.align fields**
- [ ] **No color.conditions unless user explicitly requests thresholds**
- [ ] visualization.result.by is empty array
- [ ] Schema compliance verified

## Generation Workflow

1. **Input Analysis**
   - Identify data source type
   - Extract user-provided values
   - Note missing fields for defaults
   - Check for explicit threshold/color requests
   - Check for explicit entity/severity requests (policy)

2. **Single Source Validation**
   - Ensure only 1 data source requested
   - Reject if multiple sources specified

3. **Type-Specific Validation**
   - Apply appropriate constraints
   - Validate data point limits
   - Check aggregator constraints
   - Validate entity type constraints (only if user requests)
   - Check layout requirements

4. **Default Application**
   - Fill missing required fields
   - Apply type-specific defaults
   - **Apply mandatory style defaults: font.size="small", text.align="left"**
   - **For flow: ensure font.size and text.align always present**
   - **For policy: apply Pie layout, no severity, no entity.type by default**
   - Use global defaults as fallback

5. **Schema Validation**
   - Validate against JSON schema
   - Check enum constraints
   - Verify required field presence
   - Validate mandatory defaults applied

6. **Output Generation**
   - Generate compliant JSON
   - Include all required properties
   - Format consistently

Remember: **Complete structure integrity + all validation rules compliance + mandatory defaults application is non-negotiable**. Violating any constraint results in generation failure. 