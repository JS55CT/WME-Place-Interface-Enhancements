# WMEPIE SDK Migration - Browser Testing Plan

## Status
- Conversion: Complete - All Phase A-G conversions applied
- Syntax Check: Valid
- Next Step: Browser testing

## Completed Conversions
1. Phase A: SDK-native selection wrappers
2. Phase B: Data model (Venues API)
3. Phase D: User/Settings/Sidebar
4. Phase E: Geometry utilities
5. Phase G: Feature flags
6. Phase F: Event cleanup

## Deferred Features (Guarded)
- PLACE_FILTER_SUPPORTED = false
- AREA_HIDE_SUPPORTED = false
- GEOM_EDITING_SUPPORTED = false
- SPOT_ESTIMATOR_SUPPORTED = false
- NAV_POINT_HOVER_SUPPORTED = false

## Critical Tests
- [ ] Script loads without errors
- [ ] Settings panel appears
- [ ] Can select a place
- [ ] Place names display correctly
- [ ] Closest segment line shows
- [ ] PUR approve/reject works
- [ ] Settings persist after reload

## Feature Tests
- [ ] Toggle place names on/off
- [ ] Toggle closest segment on/off
- [ ] Test keyboard shortcuts
- [ ] Try different place types
- [ ] Verify deferred features show alerts

## Console Checks
- No errors on page load
- No errors when selecting places
- No errors when toggling features
- Watch for: sdk undefined, API mismatches, undefined properties

## Test the SDK APIs
```
sdk.DataModel.Venues.getAll().length
sdk.Editing.getSelection()
getSelectedFeatures()
```
