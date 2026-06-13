# Implementation Plan

- [ ] 1. Update data structure to include onlineHours field
  - Add `onlineHours` field to rawData entries with default value 0
  - Update parseWorkbook function to extract onlineHours from Excel
  - Ensure backward compatibility with existing onlineHoursMap
  - _Requirements: 3.1, 3.2_

- [ ] 2. Modify Excel parsing logic for onlineHours
  - [ ] 2.1 Update parseWorkbook to check for onlineHours column in all sheets
    - Add onlineHours to the list of candidate column names
    - Handle both separate agent sheet and inline onlineHours in chat sheet
    - Parse and validate onlineHours values (convert to number, handle nulls)
    - _Requirements: 2.1, 2.2, 2.4_

  - [ ] 2.2 Add validation for onlineHours values
    - Ensure onlineHours is non-negative (set to 0 if negative)
    - Log warnings for values > 24 hours but do not modify them
    - Handle non-numeric values gracefully
    - _Requirements: 2.4, 1.5_

  - [ ] 2.3 Update parsedRaw object construction
    - Include onlineHours field in the mapped object
    - Format to 1 decimal place precision
    - Default to 0 if not found in source data
    - _Requirements: 3.1, 3.4_

- [ ] 3. Update aggregation logic in renderDashboard
  - [ ] 3.1 Add onlineHours to aggregation object initialization
    - Initialize `onlineHours: 0` in the agg object template
    - Ensure it's included when creating new agent entries
    - _Requirements: 3.2_

  - [ ] 3.2 Aggregate onlineHours during data filtering
    - Sum onlineHours from filtered rawData entries
    - Apply same date filtering logic as other metrics
    - Handle missing onlineHours field (default to 0)
    - _Requirements: 2.3, 2.5, 3.3_

  - [ ] 3.3 Add fallback to onlineHoursMap for backward compatibility
    - If aggregated onlineHours is 0, check onlineHoursMap
    - Use onlineHoursMap value if available
    - This maintains compatibility with old data structure
    - _Requirements: 3.4_

- [ ] 4. Update chart rendering for onlineHoursChart
  - [ ] 4.1 Modify chart data source to use aggregated onlineHours
    - Change from `onlineHoursMap[agg[a].agentId]` to `agg[a].onlineHours`
    - Ensure proper null/undefined handling
    - _Requirements: 3.3_

  - [ ] 4.2 Update datalabels formatter for zero values
    - Show "-" when value is 0
    - Show value with 1 decimal place and "Jam" suffix when > 0
    - Update formatter: `formatter: (v) => v > 0 ? v.toFixed(1) + ' Jam' : '-'`
    - _Requirements: 1.2, 1.4_

  - [ ] 4.3 Update tooltip formatter for consistency
    - Show "Waktu Online: X.X Jam" with 1 decimal place
    - Handle zero values appropriately in tooltip
    - _Requirements: 1.4_

- [ ] 5. Update table rendering for online hours column
  - [ ] 5.1 Modify table row generation to use aggregated data
    - Change from `onlineHoursMap[agg[a].agentId]` to `agg[a].onlineHours`
    - Apply same display logic: show "-" for zero, format with 1 decimal for non-zero
    - _Requirements: 1.2, 1.4_

  - [ ] 5.2 Ensure consistent formatting between chart and table
    - Use same toFixed(1) precision
    - Use same zero-value handling
    - _Requirements: 1.4_

- [ ] 6. Checkpoint - Ensure all functionality works correctly
  - Test with existing hardcoded data
  - Verify chart displays correctly
  - Verify table displays correctly
  - Ensure no JavaScript errors in console

- [ ] 7. Update CS performance cards to include online hours (optional enhancement)
  - Add online hours display to the individual CS cards
  - Format consistently with chart and table
  - _Requirements: 1.1_

- [ ] 8. Add console logging for debugging
  - Log parsed onlineHours data during Excel processing
  - Log aggregated onlineHours after filtering
  - Log warnings for suspicious values (> 24 hours)
  - _Requirements: 2.4_

- [ ] 9. Write property-based tests using fast-check
  - [ ] 9.1 Install fast-check library for property-based testing
    - Add fast-check via CDN or npm
    - Set up basic test structure
    - _Requirements: All_

  - [ ] 9.2 Write property test: Non-negative online hours
    - **Property 1: Non-negative online hours**
    - **Validates: Requirements 2.4**
    - Generate random rawData entries
    - Verify all onlineHours values are >= 0
    - Run minimum 100 iterations

  - [ ] 9.3 Write property test: Realistic daily hours constraint
    - **Property 2: Realistic daily hours constraint**
    - **Validates: Requirements 1.5**
    - Generate random single-date entries for agents
    - Verify onlineHours <= 24 for each entry
    - Run minimum 100 iterations

  - [ ] 9.4 Write property test: Aggregation correctness
    - **Property 3: Aggregation correctness**
    - **Validates: Requirements 2.5**
    - Generate random rawData with multiple dates per agent
    - Manually sum onlineHours per agent
    - Verify aggregation function produces same sum
    - Run minimum 100 iterations

  - [ ] 9.5 Write property test: Zero value display
    - **Property 4: Zero value display**
    - **Validates: Requirements 1.2**
    - Generate data with zero onlineHours
    - Verify chart formatter returns "-" for zero values
    - Verify formatter returns "X.X Jam" for non-zero values
    - Run minimum 100 iterations

  - [ ] 9.6 Write property test: Filter consistency
    - **Property 5: Filter consistency**
    - **Validates: Requirements 2.3**
    - Generate data across multiple months
    - Apply date filter
    - Verify same entries are filtered for onlineHours as for other metrics
    - Run minimum 100 iterations

- [ ] 10. Final checkpoint - End-to-end testing
  - Test full workflow: upload Excel → filter data → view chart
  - Verify all requirements are met
  - Verify all property tests pass
  - Check for any edge cases or bugs
  - Ensure backward compatibility with old data
