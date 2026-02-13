# TRACE Canvas - Comprehensive Testing Checklist
## CrawlQ Copyplatform Standards

**Test Date**: __________
**Tester Name**: __________
**Browser**: __________
**Version**: Sprint 1 Complete

---

## 🎨 **Brand & UX Compliance**

### CrawlQ Brand Colors
- [ ] Primary Blue: `#3B82F6` (buttons, links)
- [ ] Success Green: `#10B981` (success states)
- [ ] Warning Orange: `#F59E0B` (warnings)
- [ ] Error Red: `#EF4444` (errors)
- [ ] Gradients used in notifications
- [ ] Consistent spacing (Tailwind scale)

### Copy Guidelines
- [ ] Conversational tone ("Easy There!", "Slow Down!")
- [ ] Helpful, not condescending
- [ ] Action-oriented CTAs
- [ ] Error messages suggest solutions
- [ ] No technical jargon for end users

---

## ✅ **Feature Testing**

### **1. Canvas Creation & Management**

#### Create New Canvas
- [ ] Navigate to `/canvas/new`
- [ ] Page loads in < 2s
- [ ] Empty canvas displayed
- [ ] Sidebar with node types visible
- [ ] Toolbar with buttons rendered
- [ ] Title input defaults to "Untitled Workflow"

#### Save Canvas
- [ ] Create simple workflow (2 nodes)
- [ ] Click Save button
- [ ] Blue info modal appears if no title
- [ ] Auto-suggests title with date
- [ ] Success toast appears (green gradient)
- [ ] Canvas ID shown in notification
- [ ] Canvas ID appears below title input
- [ ] "Saved just now" timestamp appears
- [ ] Orange asterisk removed from Save button

#### Load Canvas
- [ ] Click Load button
- [ ] Danger modal if unsaved changes exist
- [ ] Browser prompt asks for Canvas ID
- [ ] Enter valid ID: `b99ff380-57ff-4b9e-933d-d34c46945be4`
- [ ] Success toast appears
- [ ] Canvas renders with all nodes
- [ ] Node connections preserved
- [ ] Workflow title updated in toolbar

#### Delete Canvas
- [ ] Navigate to `/canvas`
- [ ] Click trash icon on canvas card
- [ ] Danger modal confirms deletion
- [ ] Canvas disappears from list immediately
- [ ] Success toast appears
- [ ] Deleted canvas not in DynamoDB

#### List Canvases
- [ ] Navigate to `/canvas`
- [ ] All saved canvases displayed
- [ ] Cards show: thumbnail, title, date, node count
- [ ] Search bar filters by title
- [ ] Refresh button reloads list
- [ ] Auto-refreshes when page gains focus
- [ ] Empty state shown if no canvases

---

### **2. Workflow Execution**

#### Run Simple Workflow
- [ ] Add Prompt node
- [ ] Connect to Output node
- [ ] Click "Run Workflow" in toolbar
- [ ] Blue "Workflow Starting" toast appears
- [ ] Nodes show "running" status (animated pulse)
- [ ] Output node displays result
- [ ] Green success toast with TRACE score
- [ ] "Workflow Complete! 🎉" message
- [ ] Execution completes in < 10s

#### Run from Output Node
- [ ] Click "Run Workflow" button on Output node
- [ ] Same validation as toolbar button
- [ ] Same success/error toasts
- [ ] Minimizes clicks (as requested)

#### Validation Errors
- [ ] Try running empty canvas
  - [ ] Red error toast: "Empty Workflow Detected"
  - [ ] Helpful message about dragging nodes

- [ ] Try running without Output node
  - [ ] Red error toast: "Missing Output Node"
  - [ ] Tip about adding from sidebar

- [ ] Try running with unconnected Output
  - [ ] Red error toast: "Output Node Not Connected"
  - [ ] Instructions on how to connect

- [ ] Add orphaned nodes (unconnected)
  - [ ] Orange warning modal lists node names
  - [ ] "Continue anyway?" prompt
  - [ ] Orphaned nodes ignored if confirmed

---

### **3. Edge Cases & Irrational Behaviors**

#### Spam Prevention
- [ ] Click Save 5 times in 2 seconds
  - [ ] Orange warning: "Slow Down There!"
  - [ ] Shows time since last save
  - [ ] Blocks rapid saves

- [ ] Click Run 3 times rapidly
  - [ ] Orange warning: "Easy There!"
  - [ ] 2-second debounce enforced

#### Empty/Invalid Titles
- [ ] Clear title → Try to save
  - [ ] Blue modal: "Name Your Workflow"
  - [ ] Suggests dated name
  - [ ] Canceling preserves empty title

- [ ] Enter 150-character title → Save
  - [ ] Orange warning: "Title Too Long"
  - [ ] Shows character count (150/100)

#### Invalid Canvas IDs
- [ ] Try loading with empty ID
  - [ ] Red error: "Invalid Canvas ID"
  - [ ] Shows expected format

- [ ] Try loading "abc123" (too short)
  - [ ] Red error: "ID Too Short"
  - [ ] Shows entered length vs expected 36

- [ ] Try loading with spaces: "abc 123 xyz"
  - [ ] Red error: "Invalid Format"
  - [ ] Tip about copying carefully

- [ ] Try loading non-existent ID
  - [ ] Red error: "Canvas Not Found"
  - [ ] Suggests checking listing page

#### Navigation Protection
- [ ] Make changes → Click "My Canvases" link
  - [ ] Red danger modal: "Unsaved Changes Detected"
  - [ ] Warns about permanent loss
  - [ ] Can cancel or continue

- [ ] Start workflow → Click "Back to Chat"
  - [ ] Browser confirm: "Workflow is still executing"
  - [ ] Can cancel or force navigate

- [ ] Make changes → Close browser tab
  - [ ] Browser alert: "You have unsaved changes"
  - [ ] Can cancel close

#### Concurrent Operations
- [ ] Start workflow → Try to Clear canvas
  - [ ] Red error: "Workflow Still Running"
  - [ ] Blocked with explanation

- [ ] Start workflow → Try to Load canvas
  - [ ] Red error: "Workflow Still Running"
  - [ ] Tip about node status badges

#### Empty Canvas Operations
- [ ] Save empty canvas (no nodes)
  - [ ] Orange warning modal: "Save Empty Workflow?"
  - [ ] Warns can't execute without nodes

- [ ] Clear canvas with 10+ nodes
  - [ ] Red danger modal: "Clear Entire Canvas?"
  - [ ] Shows node/edge count to delete
  - [ ] Success toast after clearing

#### Large Workflows
- [ ] Add 25 nodes → Try to run
  - [ ] Orange warning: "Large Workflow Detected"
  - [ ] Warns might take 30+ seconds
  - [ ] Can cancel or continue

---

### **4. Notification System**

#### Success Notifications (Green Gradient)
- [ ] Slide in from right smoothly
- [ ] Green checkmark icon with shadow
- [ ] Title in green text
- [ ] Message clear and concise
- [ ] Details in gray box (if provided)
- [ ] Location tag at bottom
- [ ] Animated progress bar
- [ ] Auto-dismiss after 4 seconds
- [ ] Close button (X) works
- [ ] Multiple stack properly (8px gap)

#### Error Notifications (Red Gradient)
- [ ] Red X icon
- [ ] Error title in red text
- [ ] Helpful recovery message
- [ ] Technical details in monospace
- [ ] Location for debugging
- [ ] Auto-dismiss after 8 seconds
- [ ] User can close manually

#### Warning Notifications (Orange Gradient)
- [ ] Orange warning icon
- [ ] Warning title in orange text
- [ ] Explains consequence
- [ ] Suggests alternative action
- [ ] Auto-dismiss after 6 seconds

#### Info Notifications (Blue Gradient)
- [ ] Blue info icon
- [ ] Info title in blue text
- [ ] Neutral, informative tone
- [ ] Auto-dismiss after 5 seconds

#### Confirmation Modals
- [ ] Backdrop blurs/dims background
- [ ] Modal scales in smoothly
- [ ] Icon badge with gradient shadow
- [ ] Title large and bold
- [ ] Message in gray text
- [ ] Details box if provided
- [ ] Two buttons: Cancel (gray) + Confirm (colored)
- [ ] Clicking backdrop closes modal
- [ ] ESC key closes modal (optional test)

---

### **5. Canvas Interactions**

#### Node Operations
- [ ] Drag nodes from sidebar
- [ ] Drop on canvas creates node
- [ ] Click node to select (blue ring)
- [ ] Drag node to move
- [ ] Delete key removes selected node
- [ ] Double-click node opens config (if applicable)

#### Edge/Connection Operations
- [ ] Drag from output handle (right side)
- [ ] Drag to input handle (left side)
- [ ] Connection line appears
- [ ] Arrow shows direction
- [ ] Click edge to select
- [ ] Delete key removes selected edge
- [ ] Can't connect output to output
- [ ] Can't connect input to input

#### Canvas Navigation
- [ ] Scroll to zoom in/out
- [ ] Pinch to zoom (touch devices)
- [ ] Drag empty space to pan
- [ ] Minimap shows overview (if visible)
- [ ] Zoom controls work (+ / - buttons)
- [ ] Fit view button centers all nodes

---

### **6. Performance Testing**

#### Load Times
- [ ] Initial page load < 2s
- [ ] Node rendering < 500ms
- [ ] Save operation < 3s
- [ ] Load operation < 2s
- [ ] Workflow execution < 10s (simple)

#### Memory Usage
- [ ] No memory leaks after 10 saves
- [ ] No memory leaks after 10 executions
- [ ] Canvas with 50+ nodes still responsive

#### Network Requests
- [ ] Save makes 1 POST request
- [ ] Load makes 1 GET request
- [ ] List makes 1 GET request
- [ ] No unnecessary polling
- [ ] Proper error handling for 500s

---

### **7. Accessibility**

#### Keyboard Navigation
- [ ] Tab key focuses elements
- [ ] Enter activates buttons
- [ ] ESC closes modals
- [ ] Arrow keys navigate canvas (optional)
- [ ] Shortcuts work (Cmd+S for Save)

#### Screen Reader
- [ ] Buttons have aria-labels
- [ ] Notifications announced
- [ ] Form inputs labeled
- [ ] Error messages associated with inputs

#### Visual
- [ ] Color contrast meets WCAG AA
- [ ] Focus outlines visible
- [ ] Text readable at all zoom levels
- [ ] No reliance on color alone

---

### **8. Mobile Responsiveness**

#### Phone (< 768px)
- [ ] Canvas usable on small screen
- [ ] Toolbar buttons stack vertically
- [ ] Notifications not clipped
- [ ] Modals fit screen
- [ ] Touch gestures work (pinch, pan)

#### Tablet (768px - 1024px)
- [ ] Sidebar and canvas side-by-side
- [ ] Comfortable hit targets (44x44px)
- [ ] No horizontal scrolling

---

### **9. Browser Compatibility**

#### Chrome/Edge
- [ ] All features work
- [ ] Notifications render correctly
- [ ] No console errors

#### Firefox
- [ ] All features work
- [ ] Notifications render correctly
- [ ] No console errors

#### Safari
- [ ] All features work
- [ ] Notifications render correctly
- [ ] No console errors

---

### **10. Data Integrity**

#### Persistence
- [ ] Saved workflows persist after refresh
- [ ] LocalStorage maintains state
- [ ] DynamoDB has correct data
- [ ] No data corruption

#### Multi-tenant Isolation
- [ ] User A can't see User B's canvases
- [ ] User A can't load User B's Canvas ID
- [ ] 403 error if attempting cross-tenant access

#### Concurrent Edits (Future)
- [ ] Two users editing same canvas (TODO)
- [ ] Last write wins (or conflict resolution)

---

## 🐛 **Bug Reporting Template**

If you find a bug, document it here:

### Bug #1
**Title**: __________
**Severity**: Critical / High / Medium / Low
**Steps to Reproduce**:
1.
2.
3.

**Expected**: __________
**Actual**: __________
**Screenshots**: __________
**Browser**: __________
**Error Message**: __________

---

## ✅ **Sign-Off**

- [ ] All critical features tested
- [ ] No critical bugs found
- [ ] UX meets CrawlQ standards
- [ ] Copy follows brand voice
- [ ] Performance acceptable
- [ ] Ready for beta users

**Tester Signature**: __________
**Date**: __________

---

## 📊 **Test Results Summary**

**Total Tests**: _____
**Passed**: _____
**Failed**: _____
**Pass Rate**: _____%

**Critical Issues**: _____
**High Priority Issues**: _____
**Medium Priority Issues**: _____
**Low Priority Issues**: _____

**Recommendation**: APPROVE / NEEDS WORK / REJECT

---

**Next Steps**:
1. Fix any critical/high priority bugs
2. Retest failed cases
3. Deploy to staging
4. Invite beta users
5. Monitor real-world usage
