# Design Document

## Overview

This design document outlines the solution to fix the "Akumulasi Waktu Aktif Kerja Online Agen" chart which currently displays inaccurate and unrealistic online hours data. The fix will ensure data accuracy, proper handling of missing data, and consistent data structure integration with the existing dashboard system.

## Architecture

The solution follows the existing dashboard architecture pattern:

1. **Data Layer**: Raw data stored in `rawData` array, with online hours stored in `onlineHoursMap` object
2. **Aggregation Layer**: Data filtered and aggregated in `renderDashboard()` function
3. **Presentation Layer**: Chart rendered using Chart.js with proper formatting

### Current Issues

1. `onlineHoursMap` is hardcoded and disconnected from `rawData`
2. Values in `onlineHoursMap` are unrealistic (97-98 hours)
3. No validation or sanity checks on online hours values
4. No proper handling of zero values in chart display

## Components and Interfaces

### 1. Data Structure

```javascript
// Current structure (to be modified)
let rawData = [
  {
    date: "2026-06-04",
    agentId: "csakseslegal1@gmail.com",
    newChat: 2,
    activeChat: 45,
    // ... other fields
    // Missing: onlineHours field
  }
];

// Proposed structure
let rawData = [
  {
    date: "2026-06-04",
    agentId: "csakseslegal1@gmail.com",
    newChat: 2,
    activeChat: 45,
    // ... other fields
    onlineHours: 8.5  // NEW FIELD: hours worked on this date
  }
];
```

### 2. Aggregation Object

```javascript
// Current aggregation (in renderDashboard function)
const agg = {
  "CS 1": {
    agentId: "csakseslegal1@gmail.com",
    newChat: 100,
    activeChat: 200,
    // ... other aggregated fields
    // Missing: onlineHours
  }
};

// Proposed aggregation
const agg = {
  "CS 1": {
    agentId: "csakseslegal1@gmail.com",
    newChat: 100,
    activeChat: 200,
    // ... other aggregated fields
    onlineHours: 45.5  // NEW: Sum of onlineHours for filtered dates
  }
};
```

### 3. Chart Configuration

```javascript
// Chart configuration for onlineHoursChart
updateChart('onlineHoursChart', 'bar', {
  labels: agents,  // CS names
  datasets: [{
    label: 'Waktu Aktif Kerja Online',
    data: agents.map(a => agg[a].onlineHours || 0),  // Use aggregated data
    backgroundColor: '#6366f1',
    borderRadius: 6
  }]
}, {
  scales: {
    x: { grid: { display: false } },
    y: { 
      grid: { color: '#f1f5f9', borderDash: [5, 5] }, 
      title: { display: true, text: 'Waktu (Jam)' },
      beginAtZero: true  // Ensure starts at 0
    }
  },
  plugins: {
    datalabels: { 
      anchor: 'end', 
      align: 'top', 
      color: '#475569', 
      font: { weight: 'bold' }, 
      formatter: (v) => v > 0 ? v.toFixed(1) + ' Jam' : '-'  // Show "-" for zero
    },
    tooltip: { 
      callbacks: { 
        label: (ctx) => `Waktu Online: ${ctx.raw.toFixed(1)} Jam` 
      } 
    }
  }
});
```

## Data Models

### rawData Entry Model

```typescript
interface RawDataEntry {
  date: string;           // Format: "YYYY-MM-DD"
  agentId: string;        // Email of agent
  newChat: number;
  activeChat: number;
  assigned: number;
  responseTime: number;   // In minutes
  invoices: number;
  nominal: number;        // In Rupiah
  totalDuration: number;  // In minutes
  totalMessages: number;
  inv_paid: number;
  inv_unpaid: number;
  inv_expired: number;
  onlineHours: number;    // NEW: In hours, default 0
}
```

### Aggregation Entry Model

```typescript
interface AggregationEntry {
  agentId: string;
  newChat: number;
  activeChat: number;
  assigned: number;
  rtSum: number;
  rtCount: number;
  invoices: number;
  nominal: number;
  totalDurationSum: number;
  totalMessagesSum: number;
  inv_paid: number;
  inv_unpaid: number;
  inv_expired: number;
  onlineHours: number;    // NEW: Sum of hours for filtered period
}
```

## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system-essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

### Property 1: Non-negative online hours

*For any* raw data entry, the onlineHours value should be greater than or equal to zero
**Validates: Requirements 2.4**

### Property 2: Realistic daily hours constraint

*For any* single date entry for an agent, the onlineHours value should not exceed 24 hours
**Validates: Requirements 1.5**

### Property 3: Aggregation correctness

*For any* filtered dataset and agent, the aggregated onlineHours should equal the sum of onlineHours from all entries for that agent in the filtered date range
**Validates: Requirements 2.5**

### Property 4: Zero value display

*For any* agent with zero onlineHours in aggregation, the chart label should display "-" instead of a numeric value
**Validates: Requirements 1.2**

### Property 5: Filter consistency

*For any* date filter applied, the onlineHours data should be filtered using the same logic as other metrics (newChat, activeChat, etc.)
**Validates: Requirements 2.3**

### Property 6: Data structure consistency

*For any* parsed Excel data, if onlineHours column exists, it should be included in rawData entries; otherwise default to 0
**Validates: Requirements 3.1, 3.2**

## Error Handling

### 1. Missing onlineHours in Excel

**Scenario**: Excel file does not contain "Online Hours" column

**Handling**:
- Set `onlineHours: 0` for all rawData entries
- Display "-" for all agents in chart
- No error message shown to user (graceful degradation)

### 2. Invalid onlineHours values

**Scenario**: Excel contains negative or non-numeric values

**Handling**:
- Validate and convert to 0 if invalid
- Log warning to console for debugging
- Continue processing other data

### 3. Unrealistic values

**Scenario**: Online hours > 24 for a single day

**Handling**:
- Accept the value (might be cumulative from system)
- Display warning in console
- Do not cap or modify the value (preserve data integrity)

### 4. All agents have zero hours

**Scenario**: No online hours data available for any agent

**Handling**:
- Chart displays all bars at zero height
- Labels show "-" for all agents
- Chart remains visible (no empty state message)

## Testing Strategy

### Unit Tests

The implementation will include unit tests for critical functions:

1. **Data parsing tests**
   - Test parsing Excel with onlineHours column
   - Test parsing Excel without onlineHours column
   - Test handling of invalid onlineHours values

2. **Aggregation tests**
   - Test summing onlineHours across multiple dates
   - Test filtering onlineHours by date range
   - Test aggregation with zero values

3. **Formatting tests**
   - Test display of zero values as "-"
   - Test decimal precision (1 decimal place)
   - Test chart data preparation

### Property-Based Tests

Property-based tests will be implemented using the fast-check library (JavaScript property-based testing library). Each test should run a minimum of 100 iterations to ensure comprehensive coverage across random inputs.

1. **Property 1 Test: Non-negative values**
   ```javascript
   // Generate random rawData entries, verify all onlineHours >= 0
   ```

2. **Property 2 Test: Daily limit**
   ```javascript
   // Generate random single-date entries, verify onlineHours <= 24
   ```

3. **Property 3 Test: Aggregation sum**
   ```javascript
   // Generate random rawData with multiple dates per agent
   // Verify sum equals aggregation
   ```

4. **Property 4 Test: Zero display**
   ```javascript
   // Generate data with zero onlineHours
   // Verify chart formatter returns "-"
   ```

5. **Property 5 Test: Filter consistency**
   ```javascript
   // Generate data across multiple months
   // Apply filter, verify same entries filtered for all metrics
   ```

### Integration Tests

1. Test full upload flow: Excel → parse → render → display
2. Test filter changes: Date filter → reaggregation → chart update
3. Test chart interaction: Hover tooltips, label display

## Implementation Notes

### Backward Compatibility

- Existing `onlineHoursMap` will be deprecated but kept for backward compatibility
- If `onlineHours` field is not in rawData, fall back to `onlineHoursMap`
- Gradual migration: Excel parser populates both structures during transition

### Performance Considerations

- Adding `onlineHours` to rawData adds minimal memory overhead (~8 bytes per entry)
- Aggregation loop already iterates through all entries, adding one field has O(1) impact
- No significant performance degradation expected

### Data Migration

For existing deployments:
1. Update Excel templates to include "Online Hours" column
2. Backend should export online hours data in reports
3. Dashboard will handle both old (onlineHoursMap) and new (rawData) structures

### Chart Rendering Optimization

- Use Chart.js built-in zero-value handling
- Optimize label formatter to avoid repeated calculations
- Cache formatted values if needed for performance
