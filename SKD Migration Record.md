# WMEPIE Complete Feature Map & Migration Record

Last updated: 2026-03-11. Source of truth for what every feature does and how it was migrated.

---

## Settings Panel Structure (4 fieldsets + 1 section)

1. **Place Filter Panel** (`fieldPlaceFilter`) — text filter + Hide/Show Only radio
2. **Place Properties Panel** (`fieldPlacePanel`) — area size display, locator crosshair, search button, copy place, GPID tooltip, clear description, geometry mods, simplify factor
3. **New Places Panel** (`fieldNewPlaces`) — auto-edit after create, street/city from closest segment, default lock level
4. **Map Changes Panel** (`fieldMapMods`) — place names overlay, nav point hover, GLE, openPUR, photo viewer, enlarge geo handles
5. **Place Menu Customization** — 12 configurable quick-apply shortcut slots

---

## Feature Migration Table

### Infrastructure / Init

| Feature | Key Change |
| --- | --- |
| Bootstrap / ready state | `sdk.Events.on('wme-ready', ...)` via Bootstrap utility |
| Tab registration | `WazeWrap.Interface.Tab` → `sdk.Sidebar.registerScriptTab()` |
| Settings persistence | `WazeWrap.Remote.RetrieveSettings/SaveSettings` retained for cross-device sync |
| User rank | `WazeWrap.User.Rank()` → `sdk.State.getUserInfo()?.rank + 1` |
| Locale | `I18n.currentLocale()` → `sdk.Settings.getLocale().localeCode` |
| Spanish locale | `'es'` → `'es-419'`; matching uses `locale.startsWith(currentLocale)` |
| Imperial units | `W.prefs.attributes.isImperial` → `sdk.Settings.getUserSettings().isImperial` |

### Selection System

| Feature | Key Change |
| --- | --- |
| Get selected venue | `W.selectionManager.getSelectedFeatures()[0].model` → `sdk.Editing.getSelection()` + `getById` |
| Venue attributes | `venue.attributes.*` (Backbone) → direct SDK Venue properties |
| Lock rank | `venue.attributes.lockRank` → `venue.lockRank` (top-level) |
| House number | `venue.attributes.houseNumber` → `sdk.DataModel.Venues.getAddress({ venueId })` |
| Has selection | `WazeWrap.hasSelectedFeatures()` → `getSelectedFeatures().length > 0` |

Helper wrappers added: `getSelectedFeatures()`, `hasPlaceSelected()`, `getSelectedPlace()`,
`venueIsPoint()`, `venueIsParkingLot()`, `venueGetCentroid()`.

### Data Updates

| Feature | Key Change |
| --- | --- |
| Update venue properties | `new UpdateObject(venue, {...}) + actionManager.add()` → `sdk.DataModel.Venues.updateVenue(...)` |
| Update PUR | `new UpdatePlaceUpdate(...) + actionManager.add()` → `sdk.DataModel.Venues.updateVenueUpdateRequest(...)` |
| Make Primary alias | `MultiAction + 2x UpdateObject + require(...)` → single `updateVenue({ venueId, name, aliases })` |
| Hours update | `UpdateObject` with openingHours → `updateVenue({ venueId, openingHours })` |
| Clear description | `UpdateObject` → `updateVenue({ venueId, description: '' })` |
| Iterate all venues | `W.model.venues.objects` → `sdk.DataModel.Venues.getAll()` |
| require() calls | All `require('Waze/Feature/...')` / `require('Waze/Action/...')` removed |

### Event System

| Event | Before | After |
| --- | --- | --- |
| Selection changed | `WazeWrap.Events.register('selection')` | `sdk.Events.on({ eventName: 'wme-selection-changed' })` |
| Data model changed | `WazeWrap.Events.register('change:venues')` | `trackDataModelEvents` + `wme-data-model-objects-changed` |
| Zoom changed | `W.map.events.register('zoomend')` | `wme-map-zoom-changed` |
| Layer visibility | `W.map.events.register('changelayer')` | `wme-map-layer-changed` |
| Mode changed | `WazeWrap.Events.register('change:mode')` | **Deleted** — confirmed dead code |

### Place Names Overlay

| Feature | Key Change |
| --- | --- |
| Custom draw layer | `new OpenLayers.Layer.Vector` + `W.map.addLayer` → `sdk.Map.addLayer({ layerName: 'PIEPlaceNameLayer', ... })` |
| Add labels | `layer.addFeatures([new OL.Feature])` → `sdk.Map.addFeaturesToLayer(...)` |
| Remove labels | `layer.removeAllFeatures()` → `sdk.Map.removeAllFeaturesFromLayer(...)` |
| Font/style | OL StyleMap → SDK `styleRules` + `styleContext` predicates |
| Sub-options | Point names, area names, PLA names, lock level overlay, hide when places hidden, font size/color/bold/outline |

### Place Filter & Hide Area Places

| Feature | Key Change |
| --- | --- |
| Filter by name | `W.map.venueLayer.styleMap` + `OpenLayers.Filter.Comparison` → `sdk.Map.addStyleRuleToLayer` predicate |
| Hide area places | OL filter add/splice → `pieHideAreaEnabled` flag + `sdk.Map.redrawLayer({ layerName: 'venues' })` |
| State vars | `let pieFilterRegex`, `let pieFilterHideMode`, `let pieHideAreaEnabled` — module-level flags |
| Functions | `UpdatePlaceFilter()`, `ToggleHideAreaPlaces()` — both ~5 lines now |

### Nav Point Hover — Closest Segment Visualization

| Feature | Key Change |
| --- | --- |
| Hover detection | `W.map.venueLayer.getFeatureBy('renderIntent','highlight')` polled → `sdk.Events.trackLayerEvents` + `wme-layer-feature-mouse-enter/leave` |
| Stop points layer | OL Vector layer → `sdk.Map.addLayer` + `addFeaturesToLayer` (layer: `_PIE_SHOW_STOP_POINTS_LAYER`) |
| Closest segment layer | OL Vector layer → `sdk.Map.addLayer` + `addFeaturesToLayer` (layer: `_PIE_CLOSEST_SEGMENT_LAYER`) |
| Closest segment calc | `WazeWrap.Geometry.findClosestSegment()` → `turf.nearestPointOnLine()` |
| Zoom check | `W.map.getZoom()` → `sdk.Map.getZoomLevel()` |
| State removed | `highlighting`, `highlightedVenue`, `NAV_POINT_HOVER_SUPPORTED` flag all deleted |

### Geometry Editing

| Feature | Key Change |
| --- | --- |
| Rotate polygon | Custom OL transform → `sdk.Map.enablePolygonRotation()` / `disablePolygonRotation()` |
| Resize polygon | Custom OL transform → `sdk.Map.enablePolygonResize()` / `disablePolygonResize()` |
| Orthogonalize | `WazeWrap.Util.OrthogonalizeGeometry()` → native `GeoJSONOrthogonalizeGeometry()` (pure JS) |
| Simplify | OL geometry simplify → `turf.simplify()` |
| Point guard | None (crashed on Point) → `!venueIsPoint(venue)` guard; buttons hidden for points |
| Read geometry | `W.userscripts.toGeoJSONGeometry()` → `venue.geometry` (SDK already GeoJSON) |
| Write geometry | `W.userscripts.toOLGeometry()` + UpdateObject → `updateVenue({ venueId, geometry })` |
| Enlarge handles | `W.map.controls.find(...)` OL approach → CSS injection on `circle[fill="white"][r="6"]` |

### Geometry Calculations (Turf.js v7)

| Legacy | Turf Replacement |
| --- | --- |
| `WazeWrap.Geometry.findClosestSegment()` | `turf.nearestPointOnLine()` |
| Custom Haversine `calculateDistance()` | `turf.distance()` |
| Custom trig `CalculateLongOffsetGPS()` | `turf.destination()` |

All geometry stays WGS84 (degrees) natively — Mercator projection math removed from calc paths.

### openPUR — Auto-Open PUR Review

Shadow DOM structure (WME changed significantly):
`wz-alert.sidebar-alert → shadowRoot → wz-button → shadowRoot → button`

```js
setTimeout(() => {
  const wzAlert = document.querySelector('wz-alert.sidebar-alert');
  const wzBtn = wzAlert?.shadowRoot?.querySelector('wz-button');
  const btn = wzBtn?.shadowRoot?.querySelector('button') ?? wzBtn;
  if (btn) btn.click();
}, 300);  // 300ms: wme-selection-changed fires before sidebar renders
```

Data check: `sdk.DataModel.Venues.getById({ venueId }).venueUpdateRequests.length`

### Photo Viewer

| Feature | Key Change |
| --- | --- |
| Scan for photos | `W.model.venues.objects` → `sdk.DataModel.Venues.getAll()` |
| Venue data access | `venue.attributes` → direct SDK Venue properties |
| Delete image | `UpdateObject` + actionManager → `updateVenue({ venueId, images })` |

### Keyboard Shortcuts

| Feature | Key Change |
| --- | --- |
| Register | `W.accelerators.addAction` → `sdk.Shortcuts.createShortcut({ shortcutId, description, callback, shortcutKeys: combo })` |
| Format | Raw bitfield `'3,65'` → combo string `'CS+A'` — **NEVER use .raw, breaks registration** |
| Persistence | `W.accelerators` → settings object via `WazeWrap.Remote` |
| Init crash fix | `for (i=0;...)` → `for (let i=0;...)` — bare `i` caused ReferenceError, blocked ALL shortcuts |

### Map Navigation

| Feature | Key Change |
| --- | --- |
| Center map | `W.map.setCenter(OLLonLat)` → `sdk.Map.centerMapOnGeometry({ geometry })` |
| Get zoom | `W.map.getZoom()` → `sdk.Map.getZoomLevel()` |
| Set zoom | `W.map.zoomTo(level)` → `sdk.Map.setZoomLevel({ zoomLevel })` |

### Google Link Enhancer (GLE)

Initialized as: `new SDKGoogleLinkEnhancer(sdk, turf, { layerName: 'PIE - Highlight closed Places' })`
Strings wired to `I18n.t()` keys (permClosedPlace, tempClosedPlace, multiLinked, linkedToThisPlace).

### Website URL Clickable Link

On `wme-selection-changed`: wraps the website `<label>` in an `<a href>` tag. Pure DOM — no migration needed. Uses `getSelectedFeatures()[0].url`.

### House Number Trim

On `wme-selection-changed`: `.focusout` trims whitespace from `.form-control.house-number`. Pure DOM — no migration needed.

---

## Still Active Legacy (Permanent / Intentional)

| Item | Why |
| --- | --- |
| `WazeWrap.Alerts.error(...)` | User decision; no SDK styled alert |
| `WazeWrap.Remote.RetrieveSettings/SaveSettings` | Cross-device sync; keep intentionally |
| `WazeWrap.Interface.ShowScriptUpdate(...)` | Update notification; keep intentionally |

**Zero active `W.` calls remain in any live code path.**

### Create PLA from Map Problem — FULLY REMOVED (2026-03-12)

Entire feature chain deleted: `MPLayerChanged`, `MarkerClick`, `GetMarkerType`, `MO_MPLayer`
MutationObserver, and all `W.map.getLayerByName('mapProblems')` calls.
`createPLAFromMP` function body was already gone; orphaned call + button UI removed with it.
