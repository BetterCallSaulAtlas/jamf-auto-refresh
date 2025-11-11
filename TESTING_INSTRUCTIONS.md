# Testing Instructions for Duplication Fixes

## Quick Test (2 minutes)

1. **Install the updated script** (v2.1.1) in your browser extension
2. **Open Jamf Pro** in your browser
3. **Navigate between pages** (Computers → Devices → Settings → Back to Computers)
4. **Open browser console** (F12)
5. **Look for this message** after each navigation:
   ```
   [Jamf Auto-Refresh] Already loaded in this window session, aborting
   ```
6. **Verify**: Only ONE floating widget is visible on screen

✅ **PASS**: If you see the message and only one widget
❌ **FAIL**: If you see multiple widgets or no console message

---

## Detailed Test (5 minutes)

### Step 1: Fresh Start
1. Open a new browser tab
2. Navigate to any Jamf Pro page (e.g., `https://yourinstance.jamfcloud.com/legacy/computers.html`)
3. Wait for the page to fully load
4. Verify the floating widget appears

**Expected Console Output:**
```
[Jamf Auto-Refresh] Using DEFAULT_DOMAIN_CONFIG: ...
[Jamf Auto-Refresh] Matched configuration: ...
[Jamf Auto-Refresh] Floating window created
```

### Step 2: SPA Navigation Test
1. Click on different menu items in Jamf's sidebar:
   - Computers
   - Mobile Devices
   - Users
   - Settings
2. Watch the console after each navigation

**Expected Console Output (after each navigation):**
```
[Jamf Auto-Refresh] Already loaded in this window session, aborting
```

**Visual Check:**
- ✅ Only ONE widget remains visible
- ✅ Widget position is preserved
- ✅ Timer continues counting
- ✅ No visual glitches or duplicates

### Step 3: Widget Functionality Test
1. **Drag the widget** - should move smoothly
2. **Change the interval** - dropdown should work
3. **Click toggle** - should pause/resume
4. **Open settings** - modal should appear
5. **Navigate to another page** - widget should persist with same settings

### Step 4: Protection Flags Test
Open browser console and run:
```javascript
console.log({
  loaded: window.__jamfAutoRefreshLoaded__,
  historyPatched: window.__jamfAutoRefreshHistoryPatched__,
  dragListeners: window.__jamfAutoRefreshDragListeners__
});
```

**Expected Output:**
```javascript
{
  loaded: true,
  historyPatched: true,
  dragListeners: true
}
```

### Step 5: Widget Count Test
Run this in console:
```javascript
document.querySelectorAll('[id*="cc-auto-refresh"]').length
```

**Expected Output:** `1` (exactly one widget)

---

## Advanced Testing with Test Page

1. Open `tmp_rovodev_test_duplication.html` in your browser
2. Ensure the Jamf Auto Refresh script is enabled (it matches `*://*/*`)
3. Use the test buttons:
   - **Simulate SPA Navigation** - triggers history events
   - **Force Script Reload** - checks protection flags
   - **Check for Duplicates** - counts widgets and verifies flags
4. Monitor the stats panel and log output

---

## Performance Testing

### Memory Leak Check
1. Open Chrome DevTools → Performance tab
2. Click "Record" (circle icon)
3. Navigate between Jamf pages 10-15 times
4. Stop recording
5. Check the memory graph

**Expected:** Relatively flat line (no continuous growth)
**Problem:** Steadily increasing memory (indicates leak)

### Event Listener Count
1. Open Chrome DevTools → Performance Monitor
2. Add "Event listeners" metric
3. Navigate between pages several times
4. Watch the event listener count

**Expected:** Stable count (increases once, then stays constant)
**Problem:** Count increases with each navigation

---

## Regression Testing

Verify these features still work correctly:

- [ ] Auto-refresh timer counts down correctly
- [ ] Pause/Resume toggle works
- [ ] Refresh interval can be changed
- [ ] "Refresh Now" button works
- [ ] Widget can be dragged and position is saved
- [ ] Settings modal opens and saves configurations
- [ ] Domain matching works (script only runs on configured domains)
- [ ] Path filtering works (include/exclude patterns)
- [ ] Timer resets on manual navigation
- [ ] Session statistics are tracked correctly

---

## Browser Compatibility Testing

Test on these combinations:

- [ ] Chrome + Tampermonkey
- [ ] Chrome + Violentmonkey  
- [ ] Firefox + Tampermonkey
- [ ] Firefox + Greasemonkey
- [ ] Edge + Tampermonkey
- [ ] Safari + Userscripts (if applicable)

---

## Known Good Console Output Pattern

```
Script started: [uuid]
[Jamf Auto-Refresh] Using DEFAULT_DOMAIN_CONFIG: ...
[Jamf Auto-Refresh] Matched configuration: ...
[Jamf Auto-Refresh] Floating window created

[User navigates to another page]

Script started: [uuid]
[Jamf Auto-Refresh] Already loaded in this window session, aborting
```

---

## Troubleshooting

### Issue: Widget doesn't appear at all
- Check if domain is in the configuration
- Check browser console for errors
- Verify script is enabled in extension

### Issue: Multiple widgets still appear
- Hard refresh the page (Ctrl+Shift+R)
- Clear browser cache
- Check if you have multiple versions of the script installed
- Verify version is 2.1.1 or higher

### Issue: "Already loaded" message but no widget
- The DOM might have been cleared by Jamf
- This is expected behavior - the widget should reappear on next full page load
- The protection is working correctly

### Issue: Console shows script errors
- Check if localStorage is enabled
- Check if there are conflicting scripts
- Report the error with full stack trace

---

## Success Criteria

✅ All tests pass when:
1. Only one widget appears regardless of navigation
2. Console shows "Already loaded" message on subsequent navigations
3. All three protection flags are `true`
4. Memory usage remains stable
5. Event listener count doesn't grow
6. All functionality continues to work correctly

---

## Reporting Issues

If you find duplication issues after applying fixes, please provide:

1. Browser and extension versions
2. Jamf Pro version/URL pattern
3. Full console output from page load through navigation
4. Screenshot showing duplicate widgets
5. Output of protection flags check
6. Steps to reproduce

Create an issue at: https://github.com/BetterCallSaulAtlas/jamf-auto-refresh/issues
