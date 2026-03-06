# Turf.js Integration Strategy

## Current Status ✅

### Foundation Complete (Commit 3168be5)
- ✅ Turf.js CDN added to @require
- ✅ 10 Turf helper functions created
- ✅ Syntax validated

### Version: 2025.06.04.07

## Architecture Decision

**Hybrid Approach** (Optimal for this migration):
- **Display Layer**: Keep OpenLayers geometry (layer system)
- **Calculations**: Use Turf.js (all math operations)
- **Conversion**: Only when necessary

### Why This Approach?

1. **OpenLayers geometries** are tied to the layer visualization system:
   - PIEPlaceNameLayer (display names)
   - showStopPointsLayer (draw lines to nav points)
   - closestSegmentLayer (draw closest segment line)
   - These are OL Vector layers that require OL.Feature.Vector

2. **Turf.js advantages** for calculations:
   - Standard GeoJSON format
   - Better documented than OL geometry
   - More geometric operations (simplify, buffer, point-in-polygon)
   - Smaller library footprint for what we need

## Phase 4: Turf Integration Plan

### Step 1: Replace WazeWrap.Geometry Operations (7 operations)

| Operation | Current | Replacement | File Lines |
|-----------|---------|------------|-----------|
| findClosestSegment | WazeWrap.Geometry | turf.nearestPoint | 2182, 2247, 2806 |
| calculateDistance | WazeWrap.Geometry | geometryDistance | 3488, 3504 |
| CalculateLongOffsetGPS | WazeWrap.Geometry | turf.destination | 3776, 3781 |
| simplify | OL.simplify | geometrySimplify | 3056 |
| getCentroid | OL geometry method | geometryCentroid | Various |

### Step 2: Replace OpenLayers Geometry Calculations (12 uses)

Operations that calculate but don't display:
- Getting centroids → use geometryCentroid helper
- Getting bounds → use geometryBounds helper
- Distance calculations → use geometryDistance helper
- Simplification → use geometrySimplify helper

### Step 3: Keep OpenLayers for Display (21+ uses)

These create geometry just before adding to layers - leave as-is:
- `new OpenLayers.Geometry.Point` for layer display
- `new OpenLayers.Feature.Vector` for layer features
- `layer.addFeatures([olFeature])`

### Step 4: Guard Deferred Operations (Already Done)

Already guarded with feature flags:
- ✅ Geometry editing (OrthogonalizePlace, SimplifyPlace)
- ✅ Spot estimator (createRegularPolygon)
- ✅ Nav point hover effects

## Implementation Order

1. **Replace findClosestSegment** (3 uses)
   - Currently: WazeWrap.Geometry.findClosestSegment(point, ...)
   - New: turf.nearestPoint + SDK segments API
   - Impact: Closest segment line display

2. **Replace calculateDistance** (2 uses)
   - Currently: WazeWrap.Geometry.calculateDistance(geom.components)
   - New: geometryLength or geometryDistance
   - Impact: Spot estimator calculations

3. **Replace CalculateLongOffsetGPS** (2 uses)
   - Currently: WazeWrap.Geometry.CalculateLongOffsetGPS(offset, lon, lat)
   - New: turf.destination([lon, lat], offsetKm, bearing)
   - Impact: Place copying with offset

4. **Replace geometry methods** (Ongoing)
   - Replace `.getCentroid()` calls with geometryCentroid()
   - Replace `.getBounds()` calls with geometryBounds()
   - Replace `.simplify()` calls with geometrySimplify()

## Known Constraints

### Cannot Replace (Yet):
1. **W.geometryEditing** operations
   - Used in OrthogonalizePlace, SimplifyPlace
   - Already guarded with GEOM_EDITING_SUPPORTED = false
   - No direct SDK equivalent

2. **WazeWrap.Geometry.findClosestSegment**
   - Needs access to all road segments
   - May need to stay or use SDK Segments API if available
   - Check with SDK documentation

3. **OpenLayers layer system**
   - Core to display architecture
   - No SDK equivalent yet (Layer API exists but different)
   - Keep as-is for now

## Testing Strategy

After each replacement:
1. Syntax check: `node --check WMEPIE.js`
2. Browser test: Check affected feature works
3. Verify calculations are correct
4. No performance regression

## Success Criteria

- ✅ All WazeWrap.Geometry operations replaced
- ✅ All geometric calculations use Turf
- ✅ Layer display still works
- ✅ No new errors in console
- ✅ Features work as before

## Code Example: Before/After

### Before (OpenLayers + WazeWrap)
```javascript
let closestSeg = WazeWrap.Geometry.findClosestSegment(olPoint, false, false);
let distance = WazeWrap.Geometry.calculateDistance(geom.components);
let centroid = geometry.getCentroid();
```

### After (Turf + SDK)
```javascript
let closestSeg = turf.nearestPoint([lon, lat], allSegmentsGeoJSON);
let distance = geometryLength(geometry); // in km
let centroid = geometryCentroid(geometry); // {lon, lat}
```

## File Locations to Update

1. DisplayPlaceNames (line 2502+) - Already using helpers
2. drawNavPointClosestSegmentLines (line 2087+) - Uses findClosestSegment
3. Photos_zoom & CenterOnPlace (line 1872+) - Uses geometryCentroid
4. Spot estimator (line 3300+) - Uses distance/offset
5. Geometry editing functions (guarded - skip for now)

## Notes

- Turf.js is 40KB minified - acceptable dependency
- No breaking changes to existing code
- Gradual replacement allows incremental testing
- Can defer difficult operations and revisit later

