# Recent SDK Migration Fixes (Session 2)

## Version: 2025.06.04.06

### Issues Fixed

#### 1. **DisplayPlaceNames not showing** ✅
- **Root Cause**: W.map.getZoom() → SDK needs sdk.Map.getZoomLevel()
- **Fix**: Replaced all W.map.getZoom() with sdk.Map.getZoomLevel()
- **Also**: Removed redundant getById() call in loop - use venues directly

#### 2. **Area place selection crashes** ✅
- **Root Cause**: InsertGeometryMods calls .getOLGeometry() on SDK venues
- **Fix**: Added guard - if (!GEOM_EDITING_SUPPORTED) return;
- **Impact**: Geometry editing UI won't show (deferred feature)

#### 3. **getOLGeometry utility functions fail** ✅
- **Fixes**:
  - onScreen() - use isInMapExtent() helper
  - CenterOnPlace() - use venueGetCentroid() helper
  - Both now work with SDK geometry

#### 4. **Map centering not working** ✅
- **Root Cause**: W.map.setCenter() API changed
- **Fixes**:
  - W.map.setCenter([x, y], zoom) → sdk.Map.setMapCenter({ lonLat: { lon, lat }, zoomLevel })
  - W.map.getOLMap().zoomTo(17) → sdk.Map.setZoomLevel(17)

### Commits Applied

| Commit | Message |
|--------|---------|
| 30cd2ce | Fix area place selection guards |
| 8da966c | Fix DisplayPlaceNames zoom API |
| 204afc8 | Replace W.map.setCenter with SDK API |

### What to Test Now

**Critical (Must Work):**
1. [ ] Toggle "Show Place Names" - names should appear
2. [ ] Zoom in/out - names appear at correct zoom levels
3. [ ] Lock level display - shown with "(L1)" format when enabled
4. [ ] Select area place - no crash, works normally
5. [ ] Click photo - zoom to location works

**Should Work:**
6. [ ] Different place types - points vs areas
7. [ ] Place filtering - name filter works
8. [ ] Settings persist - reload page

**Expected (Features Deferred):**
9. [ ] Geometry editing buttons - won't appear (guarded)
10. [ ] Geometry mods - won't show (guarded)

### Remaining Known Issues

**Not Yet Fixed:**
- W.map.addLayer() calls (lines 460, 464, 475, 479, etc.)
  - May still work during transition period
  - Will need Phase C implementation for full SDK layer API

- WazeWrap.Model.getObjectModel() in place filter (guarded)
  - Already blocked by PLACE_FILTER_SUPPORTED = false

- W.geometryEditing integration
  - Already blocked by GEOM_EDITING_SUPPORTED = false

### Next Steps After Testing

If place names still don't show:
1. Check browser console for JavaScript errors
2. Verify zoom level with: sdk.Map.getZoomLevel()
3. Verify venues loading with: sdk.DataModel.Venues.getAll().length
4. Check if layer is visible: PIEPlaceNameLayer.getVisibility()

If other features broken:
- Report specific error message from console
- Note which feature/button triggered it
- Include JavaScript error stack trace

