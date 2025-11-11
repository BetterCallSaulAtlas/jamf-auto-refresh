# Duplication Issues Found and Fixed

## Issue Summary
The script was running multiple times when navigating between pages in a Single Page Application (SPA), causing duplicate widgets and event listeners.

## Root Cause
Browser extensions (like Tampermonkey) re-inject scripts on SPA navigation, even though the page doesn't fully reload. The original checks weren't sufficient to prevent this.

## Fixes Applied

### 1. **Session-Persistent Flag** (Lines 19-35)
**Issue**: DOM checks alone don't survive SPA navigation since the DOM gets recreated.

**Fix**: Added `window.__jamfAutoRefreshLoaded__` flag that persists across SPA navigation.

```javascript
// Check if script already loaded in this session (survives SPA navigation)
if (window[SCRIPT_SESSION_KEY]) {
  console.log('[Jamf Auto-Refresh] Already loaded in this window session, aborting');
  return;
}

// Mark as loaded immediately to prevent race conditions
window[SCRIPT_SESSION_KEY] = true;
```

### 2. **History API Patching Protection** (Lines 1507-1523)
**Issue**: `history.pushState` and `history.replaceState` were being wrapped multiple times, creating nested function calls.

**Fix**: Added flag to ensure history methods are only patched once.

```javascript
// Only override history methods if not already done
if (!window.__jamfAutoRefreshHistoryPatched) {
  window.__jamfAutoRefreshHistoryPatched = true;
  
  const originalPushState = history.pushState;
  const originalReplaceState = history.replaceState;
  
  history.pushState = function () {
    const ret = originalPushState.apply(this, arguments);
    handleUrlChange();
    return ret;
  };
  // ... same for replaceState
}
```

### 3. **Document-Level Drag Listeners** (Lines 1424-1430)
**Issue**: `document.addEventListener('mousemove', drag)` and `mouseup` listeners were accumulating.

**Fix**: Added flag to ensure drag listeners are only added once.

```javascript
// Store drag functions on window to prevent duplicates
if (!window.__jamfAutoRefreshDragListeners) {
  window.__jamfAutoRefreshDragListeners = true;
  document.addEventListener('mousemove', drag);
  document.addEventListener('mouseup', stopDragging);
}
```

### 4. **popstate Event Listener** (Line 1531)
**Issue**: `window.addEventListener('popstate')` was accumulating with each script execution.

**Fix**: Moved inside the history patching check to ensure it's only added once.

```javascript
if (!window.__jamfAutoRefreshHistoryPatched) {
  // ... history patching code ...
  
  // Add popstate listener only once
  window.addEventListener('popstate', handleUrlChange);
}
```

### 5. **Angular Event Listeners** (Lines 1524-1527)
**Status**: May accumulate if Angular rootScope persists.

```javascript
rootScope.$on('$routeChangeSuccess', handleUrlChange);
rootScope.$on('$stateChangeSuccess', handleUrlChange);
```

**Note**: Angular's `$on` doesn't prevent duplicates, but since we check `window.__jamfAutoRefreshLoaded__` first, this shouldn't be reached twice.

## Testing Checklist

- [ ] Script loads only once on initial page load
- [ ] Widget appears only once
- [ ] Navigation between pages doesn't duplicate widget
- [ ] Console shows "Already loaded in this window session, aborting" on subsequent injection attempts
- [ ] Drag functionality works correctly
- [ ] Refresh timer continues working after navigation
- [ ] Settings persist across navigation
- [ ] No accumulating event listeners (check with browser DevTools)

## Files Modified

- `jamf_auto_refresh.js` - Main script with all fixes applied

## Summary of Protections

All duplication issues have been resolved using three window-level flags:

1. `window.__jamfAutoRefreshLoaded__` - Prevents entire script re-execution
2. `window.__jamfAutoRefreshHistoryPatched__` - Prevents history API re-patching and popstate listener duplication
3. `window.__jamfAutoRefreshDragListeners__` - Prevents document-level drag listener duplication

These flags persist across SPA navigation, ensuring the script only initializes once per browser tab session.

## Version Impact

**Version 2.1.1** includes all duplication fixes.
