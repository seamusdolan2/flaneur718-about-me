# WordPress & Uncode — Technical Master Reference

**Last Updated:** February 2026  
**Project:** Flaneur 718 About Me Page

A living reference for WordPress and Uncode theme techniques consolidated from actual project implementation. All patterns have been tested and proven in production.

---

## Environment

- **WordPress:** 6.9.1
- **Theme:** Uncode v2.11.3
- **Child theme:** Not required for custom CSS/JS injection
- **Project prefix:** `fl718-` (use your own prefix to avoid conflicts)

---

## Where Custom Code Goes

**WordPress Dashboard → Uncode → Theme Options → Utility → CSS & JS**

- **Custom CSS** → paste in the "CSS" text box
- **Custom JavaScript** → paste in the "JavaScript" text box
- Click **SAVE CHANGES** after pasting

**CRITICAL:** CDN scripts (like GSAP) go in the JavaScript box BEFORE your custom code

---

## Implementation Approach

1. Create a new WordPress page for the project
2. Build with semantic HTML
3. Style entirely through custom CSS/JS injection (no child theme needed)
4. Use a blank or full-width page template if available
5. **Prefix ALL CSS classes** (e.g. `fl718-`, `dd-`) to avoid theme conflicts
6. Add project-specific body class in Page Options

---

## Uncode Page Options (Content tab)

For full-width, minimal chrome:

- **Content Width:** Full
- **Show Media, Show Share, Show Tags:** No
- **After Content** (Author Profile, Related Posts): No
- **Body CSS class:** Use a project-specific class (e.g. `fl718-about-page`) for scoping

---

## Uncode CSS Gotchas

### Layout Width

Uncode's global Layout Width (e.g. 1400px) constrains all content. To break out, use overrides like `width: 100% !important` on your page body class wrapper.

### Full-Bleed Sections

The `100vw` + `left: 50%` + `translateX(-50%)` trick often breaks inside Uncode's container hierarchy. Prefer `max-width` + `margin: auto` for contained full-width sections.

**Full-width breakout technique (when text/elements are trapped in centered containers):**

```css
.fl718-fullwidth-element {
  width: 100vw !important;
  max-width: none !important;
  left: 50%;
  margin-left: -50vw;
  margin-right: -50vw;
  position: relative;
}
```

**For single-line text spanning full width:**
```css
.fl718-fullwidth-text {
  width: 100vw !important;
  left: 50%;
  margin-left: -50vw;
  margin-right: -50vw;
  position: relative;
  white-space: nowrap;  /* Prevent wrapping */
  overflow: visible;     /* Prevent clipping */
  text-align: center;    /* Center the text */
}
```

### Overflow Issues

- **Never mix** `overflow-x: hidden` with `overflow-y: visible` on the same element — the spec forces the visible axis to `auto` and can create a scrollbar
- Set `overflow: visible` on Uncode wrappers
- Use `overflow-x: hidden` only on `body.your-page-class` and your main content wrapper

### Double Scrollbar (Extra Inner Scrollbar)

**Cause:** Uncode/WordPress often gives a main content wrapper `overflow-y: auto` or `overflow: auto` (sometimes with a fixed height), so you get the browser scrollbar plus a second scrollbar on an inner div.

**Fix:** Scope to your page body class (e.g. `body.fl718-about-page`) and set all wrappers to:

```css
body.fl718-about-page .post-wrapper,
body.fl718-about-page .uncont,
body.fl718-about-page .row-container,
body.fl718-about-page .row-parent,
body.fl718-about-page #wrapper,
body.fl718-about-page .wrapper {
  overflow: visible !important;
  overflow-x: visible !important;
  overflow-y: visible !important;
  max-height: none !important;
  height: auto !important;
}
```

**If it persists:**
1. DevTools → inspect the scrollable area
2. Find the element with `overflow: auto` or `overflow-y: scroll/auto` in Computed
3. Note its class or ID
4. Add that selector to your overflow-override block
5. Re-paste CSS, SAVE in Uncode → Theme Options → Utility → CSS & JS

### Overrides

Target `#wrapper` and `.wrapper` in addition to standard Uncode class names when fixing overflow or width.

### Typography

Font sizes and colors in hero/custom sections often need `!important` to override Uncode's defaults.

---

## GSAP ScrollTrigger Integration

**What it is:** GSAP (GreenSock Animation Platform) is a JavaScript animation library. ScrollTrigger is a plugin that ties animations to scroll position.

### CDN Links (Latest Stable v3.12.5)

Add these to **Uncode → Theme Options → Utility → CSS & JS** (JavaScript box) **BEFORE** your custom animation code:

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/gsap.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/ScrollTrigger.min.js"></script>
```

### Register ScrollTrigger

At the top of your animation code:

```javascript
gsap.registerPlugin(ScrollTrigger);
```

### Basic ScrollTrigger Pattern

```javascript
gsap.to('.fl718-element', {
  opacity: 1,
  y: 0,
  scrollTrigger: {
    trigger: '.fl718-element',     // Element that triggers the animation
    start: 'top 80%',               // When top of trigger hits 80% down viewport
    end: 'bottom 60%',              // When bottom of trigger hits 60% down viewport
    scrub: 1,                       // Smooth scrubbing (tied to scroll position)
    markers: false,                 // Set to true for debugging (shows start/end lines)
  }
});
```

### Scrub Values

- `scrub: true` → Animations tied to scroll, no smoothing
- `scrub: 1` → 1 second smoothing delay (recommended for smooth feel)
- `scrub: 2` → 2 seconds smoothing (very smooth, can feel sluggish)
- No scrub → Animation plays once when triggered (not tied to scroll position)

### Start/End Positions

Format: `"[trigger position] [viewport position]"`

Examples:
- `start: 'top top'` → Trigger when top of element hits top of viewport
- `start: 'top 80%'` → Trigger when top of element hits 80% down viewport
- `start: 'center center'` → Trigger when center of element hits center of viewport
- `end: 'bottom top'` → End when bottom of element hits top of viewport

### Common GSAP Animations

```javascript
// Fade in
gsap.from('.element', { opacity: 0, duration: 1 });

// Slide up
gsap.from('.element', { y: 50, opacity: 0, duration: 1 });

// Color transition
gsap.to('.element', { backgroundColor: '#ff0000', duration: 1 });

// Blur to sharp
gsap.to('.element', { filter: 'blur(0px)', duration: 1 });

// Scale up
gsap.to('.element', { scale: 1.1, duration: 0.3 });

// Horizontal scroll (vertical scroll drives horizontal movement)
gsap.to('.film-strip', {
  x: -maxScroll,
  ease: 'none',
  scrollTrigger: {
    trigger: '.film-section',
    start: 'top top',
    end: '+=3000',
    scrub: 1,
    pin: true,
    anticipatePin: 1,
  }
});
```

### **CRITICAL: GSAP Cannot Target CSS Pseudo-elements**

❌ **This will NOT work:**
```javascript
gsap.to('.element::after', { opacity: 1 });
// Error: GSAP target .element::after not found
```

✅ **Solution: Use real DOM elements instead:**

```html
<!-- HTML -->
<div class="element">
  <span class="element-decoration"></span>
  Content
</div>
```

```css
/* CSS - style it like a pseudo-element */
.element-decoration {
  position: absolute;
  /* ...positioning */
}
```

```javascript
// JavaScript - GSAP can now target it
gsap.to('.element-decoration', { opacity: 1 });
```

**Why:** Pseudo-elements (`::before`, `::after`) are rendered by CSS but aren't real DOM nodes, so JavaScript cannot access them.

---

## IntersectionObserver API

**When to use:** One-time entrance effects, triggers that don't need scroll scrubbing

**When NOT to use:** Animations that should be tied to scroll position (use GSAP ScrollTrigger instead)

### Basic Pattern

```javascript
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      entry.target.classList.add('visible');
      observer.unobserve(entry.target); // Stop observing after trigger
    }
  });
}, {
  threshold: 0.3,  // Trigger when 30% visible
  rootMargin: '0px' // Offset from viewport edge
});

// Observe elements
document.querySelectorAll('.fl718-animate-on-scroll').forEach(el => {
  observer.observe(el);
});
```

### Threshold Values

- `threshold: 0` → Trigger as soon as 1px is visible
- `threshold: 0.5` → Trigger when 50% visible
- `threshold: 1.0` → Trigger only when 100% visible
- `threshold: [0, 0.5, 1.0]` → Trigger at multiple points

### Root Margin

- `rootMargin: '0px'` → Default (viewport edge)
- `rootMargin: '-100px'` → Trigger 100px before entering viewport
- `rootMargin: '100px'` → Trigger 100px after entering viewport

---

### Common IntersectionObserver Issues & Fixes

**Discovered:** February 15, 2026 (Chapter 4 Typewriter Bug)

#### Problem 1: Observer Never Fires (DOMContentLoaded Wrapper)

**Symptoms:**
- Element has animation CSS ready (cursor blinking)
- JavaScript code exists and looks correct
- But animation never triggers when scrolling

**Bad Pattern:**
```javascript
// ❌ Problem: Script at end of <body>, DOMContentLoaded already fired
document.addEventListener('DOMContentLoaded', () => {
  const element = document.querySelector('.my-element');
  if (element) observer.observe(element);
});
```

**Why it fails:**
- Scripts at the end of `<body>` run AFTER DOM is fully parsed
- `DOMContentLoaded` has already fired by that point
- Event listener never executes
- Observer never gets set up

**Good Pattern:**
```javascript
// ✅ Solution: Set up observer immediately (DOM is already ready)
const element = document.querySelector('.my-element');
if (element) {
  observer.observe(element);
} else {
  console.warn('Element not found for observer');
}
```

**Rule of thumb:**
- Script in `<head>` with `defer` → Use `DOMContentLoaded`
- Script at end of `<body>` → Run immediately, no wrapper needed

---

#### Problem 2: Strict textContent Check Prevents Trigger

**Symptoms:**
- Observer fires (confirmed with console.log)
- But animation still doesn't trigger
- Element appears empty but check fails

**Bad Pattern:**
```javascript
// ❌ Problem: Fails if any whitespace exists
if (entry.isIntersecting) {
  const el = document.getElementById('myElement');
  if (el && el.textContent === '') {
    // This might never run if element has whitespace
    animateElement(el);
  }
}
```

**Why it fails:**
- HTML formatting adds whitespace: `<div id="myElement">\n  </div>`
- `el.textContent` is `"\n  "` not `""`
- Check fails even though element appears empty

**Good Pattern:**
```javascript
// ✅ Solution: Use flag-based prevention instead
let triggered = false;

if (entry.isIntersecting && !triggered) {
  triggered = true;
  const el = document.getElementById('myElement');
  if (el) {
    el.textContent = '';  // Explicitly clear any whitespace
    animateElement(el);
  }
}
```

**Alternative:** Trim before checking
```javascript
if (el && el.textContent.trim() === '') {
  animateElement(el);
}
```

---

#### Problem 3: Threshold Too High (Misses Fast Scrolling)

**Symptoms:**
- Works when scrolling slowly
- Misses trigger when scrolling quickly
- Inconsistent behavior

**Bad Pattern:**
```javascript
// ❌ Problem: 20% visible might be too late
const observer = new IntersectionObserver(callback, {
  threshold: 0.2  // Requires 20% of element visible
});
```

**Why it fails:**
- Fast scrolling can skip past the 20% mark
- By the time 20% is visible, element might already be scrolling out
- High threshold = narrow trigger window

**Good Pattern:**
```javascript
// ✅ Solution: Lower threshold for more reliable triggering
const observer = new IntersectionObserver(callback, {
  threshold: 0.1,  // Trigger at 10% visible (wider window)
  rootMargin: '0px 0px -100px 0px'  // Adjust timing with margin
});
```

**Threshold Guidelines:**
- `0.05-0.1` → Best for animations that must trigger
- `0.2-0.3` → Good for lazy loading images
- `0.5+` → Use when you want element well into viewport

---

#### Problem 4: No Re-trigger Prevention

**Symptoms:**
- Animation triggers correctly first time
- But triggers again when scrolling back up and down
- Multiple animations stack or conflict

**Bad Pattern:**
```javascript
// ❌ Problem: Will re-trigger every time element enters viewport
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      entry.target.classList.add('animate');
      // ❌ Observer still watching, will trigger again!
    }
  });
});
```

**Good Pattern - Option 1: Unobserve**
```javascript
// ✅ Solution: Unobserve after first trigger
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      entry.target.classList.add('animate');
      observer.unobserve(entry.target);  // ✅ Stop watching
    }
  });
});
```

**Good Pattern - Option 2: Flag**
```javascript
// ✅ Solution: Use flag to prevent re-trigger
let triggered = false;

const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting && !triggered) {
      triggered = true;
      entry.target.classList.add('animate');
    }
  });
});
```

---

#### Problem 5: rootMargin Confusion

**Common Mistake:** Thinking negative values trigger earlier

**Reality:** Negative rootMargin triggers LATER (element needs to be further in)

**Visual Explanation:**
```
Positive rootMargin: '0px 0px 100px 0px'
┌─────────────────────┐
│    Viewport         │
│                     │
├─────────────────────┤ ← Actual viewport bottom
│  [Virtual Zone]     │ ← Triggers 100px BEFORE entering
├─────────────────────┤ ← Virtual bottom (100px past)
     Element triggers earlier

Negative rootMargin: '0px 0px -100px 0px'
┌─────────────────────┐
│    Viewport         │
│                     │
├─────────────────────┤ ← Virtual bottom (100px up)
│                     │
└─────────────────────┘ ← Actual viewport bottom
     Element must be 100px PAST top to trigger (later)
```

**Use Cases:**
- **Positive:** Lazy load images before they're visible
- **Negative:** Wait for element to be well into viewport before triggering

---

#### Problem 6: Finding the Right Timing (Iterative Refinement)

**Discovered:** February 15, 2026 (Chapter 4 Typewriter Timing)

**Symptoms:**
- Animation triggers technically work
- But timing feels wrong ("too slow" or "too fast")
- Need to find the subjective "sweet spot"

**The Challenge:**
Perfect animation timing is subjective and requires user feedback. You can't predict the ideal values without testing.

**The Iterative Process:**

**Step 1: Start Conservative**
```javascript
// Begin with a safe, late trigger
const observer = new IntersectionObserver(callback, {
  threshold: 0.1,
  rootMargin: '0px 0px -100px 0px'  // Negative = triggers later
});
```

**User Feedback:** "Too slow, starts too late when scrolling"

**Step 2: Swing to Early**
```javascript
// Try triggering much earlier
const observer = new IntersectionObserver(callback, {
  threshold: 0.05,  // More sensitive
  rootMargin: '0px 0px 300px 0px'  // Positive = triggers 300px before
});
```

**User Feedback:** "Too fast, happens before I see the section"

**Step 3: Find the Middle Ground**
```javascript
// Adjust to middle value between extremes
const observer = new IntersectionObserver(callback, {
  threshold: 0.1,
  rootMargin: '0px 0px 150px 0px'  // Half of previous attempt
});
```

**User Feedback:** "Perfect! That's the right timing"

---

**rootMargin Value Guidelines:**

| Value | Feel | Use Case |
|-------|------|----------|
| `-200px` | Very late | Element well into viewport |
| `-100px` | Late | Ensure element is visible first |
| `0px` | Default | Trigger at viewport edge |
| `50px` | Slightly early | Subtle anticipation |
| `100px` | Early | Animation ready when visible |
| `150px` | **Sweet spot** | Just before user focuses on element |
| `200px` | Very early | Aggressive pre-loading |
| `300px` | Too early | Risk triggering before visible |
| `500px+` | Way too early | Usually feels premature |

**Pro Tips:**
1. **Start conservative, iterate up** - Easier to make faster than slower
2. **Test on actual device** - Scroll speed varies on trackpad vs mouse vs mobile
3. **Get user feedback** - What feels "right" is subjective
4. **Remove delays during tuning** - `setTimeout(..., 300)` masks timing issues
5. **Consider content above** - If previous section is tall, trigger earlier
6. **Match scroll speed** - Fast-scrolling pages need earlier triggers

**Real Example (Chapter 4 Typewriter):**
```javascript
// Final tuned configuration
const chapter4Observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting && !chapter4Triggered) {
      chapter4Triggered = true;
      const el = document.getElementById('chapterNumber4');
      if (el) {
        el.textContent = '';
        typewriterEffect(el, 'CHAPTER 4', 90);
      }
      chapter4Observer.unobserve(entry.target);
    }
  });
}, {
  threshold: 0.1,  // Reliable without being too sensitive
  rootMargin: '0px 0px 150px 0px'  // Triggers just before section in view
});
```

**Why 150px worked:**
- Element starts animating ~150px before entering viewport
- By time animation completes, element is visible
- Feels smooth and anticipatory without being premature
- User's eye naturally moves to the element as it animates

**Testing Checklist:**
- [ ] Slow scroll - does it feel too eager or just right?
- [ ] Fast scroll - does it still trigger reliably?
- [ ] Medium scroll - does it feel natural?
- [ ] Does animation complete before/during visibility?
- [ ] Does it feel like the page is responding to you?

**Important:** This is UX work, not just code. The "perfect" value depends on:
- Animation duration
- Scroll speed expectations
- Visual weight of the element
- Content above and below
- Overall page pacing

**Remember:** Numbers in code don't have "feel" - test with real scrolling!

---

### IntersectionObserver Setup Template

**Complete pattern for reliable animations:**

```javascript
// 1. Flag to prevent re-triggering
let animationTriggered = false;

// 2. Create observer with optimized settings
const animationObserver = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    // 3. Check intersection AND flag
    if (entry.isIntersecting && !animationTriggered) {
      animationTriggered = true;

      const element = entry.target;

      // 4. Clear any whitespace
      if (element.textContent !== undefined) {
        element.textContent = '';
      }

      // 5. Trigger animation with delay
      setTimeout(() => {
        yourAnimationFunction(element);
      }, 300);

      // 6. Unobserve to prevent re-trigger
      animationObserver.unobserve(element);
    }
  });
}, {
  threshold: 0.1,  // Low threshold for reliability
  rootMargin: '0px 0px -100px 0px'  // Adjust timing as needed
});

// 7. Set up observer immediately (if script at end of body)
const targetElement = document.querySelector('.my-animated-element');
if (targetElement) {
  animationObserver.observe(targetElement);
} else {
  console.warn('Animation target element not found');
}

// OR: Use DOMContentLoaded if script is in <head> with defer
// document.addEventListener('DOMContentLoaded', () => {
//   const targetElement = document.querySelector('.my-animated-element');
//   if (targetElement) animationObserver.observe(targetElement);
// });
```

---

### Debugging IntersectionObserver

**Step 1: Verify Observer is Created**
```javascript
console.log('Observer created:', animationObserver);
```

**Step 2: Check Element Exists**
```javascript
const el = document.querySelector('.my-element');
console.log('Target element:', el);
```

**Step 3: Add Logging to Callback**
```javascript
const observer = new IntersectionObserver((entries) => {
  console.log('Observer fired, entries:', entries.length);
  entries.forEach(entry => {
    console.log('Entry:', {
      isIntersecting: entry.isIntersecting,
      intersectionRatio: entry.intersectionRatio,
      target: entry.target
    });
  });
});
```

**Step 4: Test Manually**
```javascript
// In browser console:
const el = document.querySelector('.my-element');
yourAnimationFunction(el);  // Should trigger animation immediately
```

**Step 5: Check for Multiple Observers**
```javascript
// If animation triggers multiple times, check for:
// - Multiple observers attached to same element
// - Observer not unobserving after trigger
// - Multiple event listeners calling observe()
```

---

## Typewriter Effects

### Basic Async Typewriter Function

```javascript
function typewriterEffect(element, text, speed = 80) {
  let index = 0;
  element.classList.add('typing');
  
  function type() {
    if (index < text.length) {
      element.textContent += text.charAt(index);
      index++;
      
      // Variable speed - slightly irregular like human typing
      const variableSpeed = speed + (Math.random() * 40 - 20);
      setTimeout(type, variableSpeed);
    } else {
      // Typing complete - remove cursor after brief pause
      setTimeout(() => {
        element.classList.remove('typing');
        element.classList.add('typed');
      }, 500);
    }
  }
  
  type();
}
```

### Typewriter with Blinking Cursor

```css
/* Blinking cursor during typing */
.fl718-title.typing::after {
  content: '|';
  animation: blink 0.7s infinite;
  margin-left: 2px;
}

/* Cursor disappears when typing complete */
.fl718-title.typed::after {
  display: none;
}

@keyframes blink {
  0%, 49% { opacity: 1; }
  50%, 100% { opacity: 0; }
}
```

### Trigger on Scroll into View

```javascript
// Trigger typewriter when element scrolls into view
const titleObserver = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting && !entry.target.classList.contains('typed')) {
      const text = entry.target.getAttribute('data-text');
      entry.target.textContent = '';
      typewriterEffect(entry.target, text, 90);
      titleObserver.unobserve(entry.target);
    }
  });
}, { threshold: 0.5 });

titleObserver.observe(document.querySelector('.fl718-chapter-title'));
```

---

## Scrapbook Animation Patterns

### Sequential "Falling In" Effect

Items drop one by one with bounce easing, like artifacts being placed in a scrapbook.

```css
/* Initial state - invisible above viewport */
.fl718-artifact {
  opacity: 0;
  transform: translateY(-60px) rotate(5deg);
  will-change: transform, opacity;
}

/* Visible state */
.fl718-artifact--visible {
  opacity: 1;
  transform: translateY(0) rotate(-2deg);
  transition: opacity 0.8s cubic-bezier(0.34, 1.56, 0.64, 1),
              transform 0.8s cubic-bezier(0.34, 1.56, 0.64, 1);
}
```

```javascript
// Sequential animation with stagger
function animateScrapbook() {
  const artifacts = document.querySelectorAll('.fl718-artifact');
  const prefersReducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
  
  if (prefersReducedMotion) {
    // Instant reveal for accessibility
    artifacts.forEach(artifact => {
      artifact.classList.add('fl718-artifact--visible');
    });
    return;
  }
  
  // Staggered animation
  artifacts.forEach((artifact, index) => {
    setTimeout(() => {
      // Add random rotation variation
      const rotation = (Math.random() * 6 - 3);
      artifact.style.setProperty('--final-rotation', `${rotation}deg`);
      artifact.classList.add('fl718-artifact--visible');
    }, index * 400); // 400ms stagger between items
  });
}
```

### Hover "Lift Off" Effect

```css
.fl718-artifact {
  transition: transform 0.4s cubic-bezier(0.34, 1.56, 0.64, 1),
              box-shadow 0.4s ease,
              z-index 0s 0s;
}

.fl718-artifact:hover {
  transform: translateY(-12px) rotate(0deg) scale(1.08);
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.35);
  z-index: 100;
  transition: transform 0.4s cubic-bezier(0.34, 1.56, 0.64, 1),
              box-shadow 0.4s ease,
              z-index 0s 0s;
}

.fl718-artifact:hover img {
  transform: scale(1.03);
}
```

---

## Polaroid Development Effects

Mimics real Polaroid instant film chemistry progression.

### Multi-Stage Development

```css
.fl718-photo__image {
  filter: brightness(0.3) contrast(0.5) saturate(0);
  will-change: filter;
}

.fl718-photo__overlay {
  position: absolute;
  top: 12px;
  left: 12px;
  right: 12px;
  bottom: 42px;
  background: rgba(255, 255, 255, 0.85);
  opacity: 1;
  transition: opacity 1s ease;
  pointer-events: none;
  z-index: 5;
}

.fl718-photo--developing .fl718-photo__image {
  animation: polaroidDevelop 4s ease-out forwards;
}

.fl718-photo--developing .fl718-photo__overlay {
  animation: overlayFade 4s ease-out forwards;
}

@keyframes polaroidDevelop {
  0% {
    filter: brightness(0.3) contrast(0.5) saturate(0);
  }
  25% {
    filter: brightness(0.6) contrast(0.7) saturate(0.3);
  }
  50% {
    filter: brightness(0.85) contrast(0.9) saturate(0.7);
  }
  75% {
    filter: brightness(0.95) contrast(0.95) saturate(0.9);
  }
  100% {
    filter: brightness(1) contrast(1) saturate(1);
  }
}

@keyframes overlayFade {
  0%, 60% {
    opacity: 0.85;
  }
  80% {
    opacity: 0.3;
  }
  100% {
    opacity: 0;
  }
}
```

### JavaScript Trigger

```javascript
const photoObserver = new IntersectionObserver((entries) => {
  entries.forEach((entry, index) => {
    if (entry.isIntersecting && !entry.target.classList.contains('fl718-photo--developing')) {
      setTimeout(() => {
        entry.target.classList.add('fl718-photo--developing');
      }, index * 1000); // Stagger each photo by 1 second
      photoObserver.unobserve(entry.target);
    }
  });
}, {
  threshold: 0.3,
  rootMargin: '-100px'
});

document.querySelectorAll('.fl718-photo').forEach(photo => {
  photoObserver.observe(photo);
});
```

---

## Film Photography Aesthetics

### Authentic Film Strip with Sprocket Holes

```html
<div class="fl718-film-strip">
  <!-- Left edge code -->
  <div class="fl718-film-strip__edge-left">KODAK PORTRA 400</div>
  
  <!-- Right edge code -->
  <div class="fl718-film-strip__edge-right">
    <span>15A</span>
    <span>22</span>
    <span>8A</span>
  </div>
  
  <!-- Frames -->
  <div class="fl718-film-strip__frames">
    <div class="fl718-frame">
      <img src="photo.jpg" alt="Frame 15A">
      <div class="fl718-frame__number">15A</div>
    </div>
  </div>
</div>
```

```css
.fl718-film-strip {
  position: relative;
  background: #000;
  padding: 30px 40px;
  border-radius: 2px;
}

/* Sprocket holes */
.fl718-film-strip::before,
.fl718-film-strip::after {
  content: '';
  position: absolute;
  top: 0;
  bottom: 0;
  width: 8px;
  background: 
    repeating-linear-gradient(
      to bottom,
      transparent 0px,
      transparent 8px,
      #fff 8px,
      #fff 16px,
      transparent 16px,
      transparent 28px
    );
}

.fl718-film-strip::before {
  left: 5px;
}

.fl718-film-strip::after {
  right: 5px;
}

/* Edge codes in orange Kodak style */
.fl718-film-strip__edge-left,
.fl718-film-strip__edge-right {
  position: absolute;
  font-family: 'Courier New', monospace;
  font-size: 8px;
  font-weight: 700;
  color: #ff6600;
  writing-mode: vertical-rl;
  transform: rotate(180deg);
  letter-spacing: 1px;
}

.fl718-film-strip__edge-left {
  left: 15px;
  top: 30px;
}

.fl718-film-strip__edge-right {
  right: 15px;
  top: 30px;
}
```

### Hand-Drawn Grease Pencil Selection Marks

Uses SVG paths for authentic wobbly, hand-drawn feel.

```html
<div class="fl718-frame fl718-frame--marked">
  <img src="photo.jpg" alt="Selected frame">
  <div class="fl718-frame__hand-drawn">
    <svg viewBox="0 0 196 196" xmlns="http://www.w3.org/2000/svg">
      <!-- Hand-drawn rectangle - intentionally wobbly -->
      <path d="M 4,10 Q 6,8 12,7 L 95,5 Q 140,4 188,7 Q 192,9 193,15 L 194,90 Q 195,140 192,186 Q 190,191 185,192 L 100,194 Q 50,195 8,193 Q 4,192 3,187 L 2,100 Q 1,50 4,10" />
    </svg>
  </div>
</div>
```

```css
.fl718-frame__hand-drawn {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 10;
  opacity: 0;
}

.fl718-frame--marked .fl718-frame__hand-drawn {
  animation: selectionDraw 0.6s ease-out forwards;
}

.fl718-frame__hand-drawn svg {
  width: 100%;
  height: 100%;
}

.fl718-frame__hand-drawn path {
  fill: none;
  stroke: rgba(255, 107, 157, 0.8);
  stroke-width: 3;
  stroke-linecap: round;
  stroke-linejoin: round;
}

@keyframes selectionDraw {
  from {
    opacity: 0;
    stroke-dasharray: 1000;
    stroke-dashoffset: 1000;
  }
  to {
    opacity: 1;
    stroke-dasharray: 1000;
    stroke-dashoffset: 0;
  }
}
```

---

## Blur-to-Focus Typography Transformations

Visual transformation without changing text content.

```css
.fl718-transform-word {
  filter: blur(3px);
  opacity: 0.4;
  color: #999;
  font-weight: 400;
  letter-spacing: 0.05em;
  display: inline-block;
  transition: all 0.8s ease-out;
}
```

```javascript
const transformWords = document.querySelectorAll('.fl718-transform-word');

transformWords.forEach((word, index) => {
  gsap.to(word, {
    filter: 'blur(0px)',
    opacity: 1,
    letterSpacing: '-0.02em',
    color: '#1a1a1a',
    fontWeight: 700,
    scrollTrigger: {
      trigger: '#transform-section',
      start: 'top 60%',
      end: 'bottom 70%',
      scrub: 1,
    },
    delay: index * 0.15, // Stagger each word
  });
});
```

---

## Color Transitions & Overlays

### Scroll-Based Color Wash Effect

```html
<section class="fl718-chapter">
  <div class="fl718-chapter__bg-overlay"></div>
  <div class="fl718-chapter__content">Content</div>
</section>
```

```css
.fl718-chapter {
  position: relative;
  background: linear-gradient(to bottom, #f0ebe5 0%, #e8a87c 100%);
}

.fl718-chapter__bg-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: linear-gradient(to bottom, #d4664a 0%, #e8a87c 100%);
  opacity: 0;
  z-index: 1;
  pointer-events: none;  /* Allow clicks through overlay */
}

.fl718-chapter__content {
  position: relative;
  z-index: 10;  /* Above overlay */
}
```

```javascript
gsap.to('.fl718-chapter__bg-overlay', {
  opacity: 0.6,
  scrollTrigger: {
    trigger: '.fl718-chapter',
    start: 'top bottom',
    end: 'bottom top',
    scrub: 1,
  }
});
```

### Photo Paper Development Effect

```css
.fl718-thesis-text {
  color: rgba(26, 26, 26, 0.1);
  filter: brightness(3) contrast(0.3);
}
```

```javascript
gsap.to('.fl718-thesis-text', {
  color: '#1a1a1a',
  filter: 'brightness(1) contrast(1)',
  scrollTrigger: {
    trigger: '.fl718-thesis-section',
    start: 'top 70%',
    end: 'center 30%',
    scrub: 1,
  }
});
```

---

## Mobile Performance & Animation Optimization

### Disable Heavy Animations on Mobile

```css
/* Desktop: Full animations */
@media (min-width: 769px) {
  .fl718-element {
    /* GSAP ScrollTrigger animations here */
  }
}

/* Mobile: Simplified or no animations */
@media (max-width: 768px) {
  .fl718-element {
    /* Static or simple fade only */
    opacity: 1;
    transform: none;
  }
  
  /* Disable blur filters (performance heavy) */
  .fl718-transform-word {
    filter: none !important;
  }
  
  /* Disable parallax */
  .fl718-parallax-element {
    transform: none !important;
  }
}
```

### Test on Real Devices

- **Chrome DevTools mobile emulation is NOT accurate** for animation performance
- Always test on actual iOS/Android devices
- Use Safari Web Inspector (iOS) or Chrome Remote Debugging (Android)

### Performance Best Practices

```css
/* Use will-change sparingly - only on animating elements */
.fl718-animating {
  will-change: transform, opacity;
}

/* Remove will-change after animation completes */
.fl718-animating.complete {
  will-change: auto;
}

/* GPU-accelerated properties (60fps) */
.fl718-smooth {
  transform: translateY(50px);  /* ✅ GPU accelerated */
  opacity: 0;                    /* ✅ GPU accelerated */
}

/* Avoid these (force repaint) */
.fl718-slow {
  top: 50px;                     /* ❌ Forces layout recalculation */
  visibility: hidden;            /* ❌ Forces repaint */
}
```

---

## Common WordPress/Uncode Debugging

### Console Errors

**"GSAP target .element not found"**
- Element doesn't exist in DOM yet
- Check if ID/class is spelled correctly
- Make sure HTML is loaded before JavaScript runs
- Use `DOMContentLoaded` if needed:

```javascript
document.addEventListener('DOMContentLoaded', () => {
  // GSAP code here
});
```

**"Uncaught ReferenceError: gsap is not defined"**
- GSAP CDN not loaded
- Check script order: GSAP CDN must come BEFORE your code
- Verify CDN URL is correct
- Check browser Network tab for 404 errors

**Animations not working:**
- Check browser console for errors
- Add `markers: true` to ScrollTrigger for debugging
- Verify element is visible (not `display: none`)
- Check z-index stacking

### DevTools Workflow

1. **Inspect element** → Check computed styles
2. **Console** → Test selectors: `document.querySelector('.my-class')`
3. **Network** → Verify CDN scripts loaded (200 status)
4. **Performance** → Check for animation jank

---

## Project Naming Conventions

### CSS Class Prefixes

Always use project-specific prefixes to avoid theme conflicts:

```css
/* ✅ Good - prefixed */
.fl718-hero { }
.fl718-chapter-1 { }
.fl718-photo-grid { }

/* ❌ Bad - generic */
.hero { }
.chapter { }
.photos { }
```

### BEM Structure (Optional)

```css
/* Block */
.fl718-film-strip { }

/* Element */
.fl718-film-strip__frame { }
.fl718-film-strip__edge-code { }

/* Modifier */
.fl718-film-strip--horizontal { }
.fl718-frame--marked { }
```

---

## WordPress Custom HTML Block Best Practices

### Image Path Options

1. **Media Library** (Recommended):
   - Upload images to WordPress Media Library
   - Use Media Library URL: `https://yourdomain.com/wp-content/uploads/2024/01/image.jpg`

2. **Hardcoded Placeholder** (During development):
   ```html
   <img src="https://placehold.co/600x400/f0ebe5/333?text=Placeholder" alt="Description">
   ```

3. **Data Attributes** (For future replacement):
   ```html
   <img data-image="hero-photo-1" src="placeholder.jpg" alt="Description">
   ```

### Semantic HTML

```html
<!-- ✅ Good - semantic and accessible -->
<article class="fl718-chapter">
  <header class="fl718-chapter__header">
    <h2 class="fl718-chapter__title">Chapter 1</h2>
  </header>
  
  <section class="fl718-chapter__content">
    <p class="fl718-paragraph">Content...</p>
  </section>
  
  <figure class="fl718-photo">
    <img src="photo.jpg" alt="Descriptive alt text">
    <figcaption>Photo caption</figcaption>
  </figure>
</article>

<!-- ❌ Bad - div soup -->
<div class="chapter">
  <div class="title">Chapter 1</div>
  <div class="content">Content...</div>
  <div class="photo">
    <img src="photo.jpg">
  </div>
</div>
```

### Accessibility Checklist

- ✅ All images have descriptive `alt` text
- ✅ Headings follow logical hierarchy (h1 → h2 → h3)
- ✅ Interactive elements have visible focus states
- ✅ Color contrast meets WCAG AA standards (4.5:1 for text)
- ✅ Animations respect `prefers-reduced-motion`

```css
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

---

## Useful Code Snippets

### Sleep/Delay Function

```javascript
function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

// Usage with async/await
async function animate() {
  element.classList.add('visible');
  await sleep(1000);  // Wait 1 second
  element.classList.add('complete');
}
```

### Check if Element Exists

```javascript
const element = document.querySelector('.fl718-element');
if (element) {
  // Element exists, safe to use
  element.classList.add('active');
} else {
  console.warn('Element not found');
}
```

### Prevent Animation Re-trigger

```javascript
let animationTriggered = false;

function myAnimation() {
  if (animationTriggered) return;
  animationTriggered = true;
  
  // Animation code...
}
```

### Random Number in Range

```javascript
// Random between min and max (inclusive)
function randomRange(min, max) {
  return Math.random() * (max - min) + min;
}

// Example: Random rotation between -5 and 5 degrees
const rotation = randomRange(-5, 5);
element.style.transform = `rotate(${rotation}deg)`;
```

### Debounce Scroll Events

```javascript
let scrollTimeout;
window.addEventListener('scroll', () => {
  clearTimeout(scrollTimeout);
  scrollTimeout = setTimeout(() => {
    // Scroll ended, do something
  }, 100);
});
```

---

## Performance Tips

### Will-Change Property

```css
/* ✅ Good - only on elements that will animate */
.fl718-animating-element {
  will-change: transform;
}

/* Remove after animation completes */
.fl718-animating-element.finished {
  will-change: auto;
}
```

**Warning:** Don't use `will-change` on too many elements (memory intensive). Only apply to elements actively animating.

### GPU Acceleration

Use `transform` and `opacity` for smooth 60fps animations:

```css
/* ✅ Good: GPU accelerated */
.fl718-element {
  transform: translateY(50px);
  opacity: 0;
  transition: transform 0.3s ease, opacity 0.3s ease;
}

/* ❌ Avoid: Forces repaint */
.fl718-element {
  top: 50px;           /* Layout recalculation */
  visibility: hidden;  /* Repaint */
  left: 100px;         /* Layout recalculation */
}
```

### Reduce Animation Complexity on Mobile

```javascript
// Detect mobile
const isMobile = window.innerWidth <= 768;

if (!isMobile) {
  // Full desktop animations
  gsap.to('.element', {
    /* complex animation */
  });
} else {
  // Simple mobile animation
  gsap.to('.element', {
    opacity: 1,
    duration: 0.3
  });
}
```

---

## Testing Checklist

### Before Launch

- [ ] Test in latest Chrome, Firefox, Safari, Edge
- [ ] Test on actual iOS device (iPhone)
- [ ] Test on actual Android device
- [ ] Test on tablet (iPad)
- [ ] Check console for errors
- [ ] Verify all images load
- [ ] Test all animations on scroll
- [ ] Check mobile hamburger menu (if applicable)
- [ ] Verify forms work (if applicable)
- [ ] Test contact page (if applicable)
- [ ] Run Lighthouse audit (Performance, Accessibility, SEO)
- [ ] Check accessibility with screen reader
- [ ] Verify meta tags and SEO
- [ ] Test page load speed
- [ ] Check HTTPS certificate
- [ ] Verify responsive breakpoints (320px, 768px, 1024px, 1440px)

### Performance Targets

- **Lighthouse Performance:** 90+
- **First Contentful Paint:** < 1.8s
- **Largest Contentful Paint:** < 2.5s
- **Time to Interactive:** < 3.8s
- **Cumulative Layout Shift:** < 0.1

---

## Resources

### Documentation

- **GSAP Docs:** https://gsap.com/docs/v3/
- **ScrollTrigger Demos:** https://gsap.com/docs/v3/Plugins/ScrollTrigger
- **IntersectionObserver:** https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API
- **Can I Use (browser support):** https://caniuse.com/
- **WebAIM Contrast Checker:** https://webaim.org/resources/contrastchecker/

### WordPress/Uncode

- **Uncode Documentation:** https://support.undsgn.com/hc/en-us
- **WordPress Codex:** https://codex.wordpress.org/

---

## Deliverables Checklist (When Handing Off)

- [ ] **CSS** → Uncode → Theme Options → Utility → CSS & JS (CSS box)
- [ ] **JavaScript** → Same location (JavaScript box)
- [ ] **HTML** → Custom HTML block in page editor
- [ ] **Images** → Uploaded to WordPress Media Library
- [ ] **Implementation guide** → Step-by-step setup documentation
- [ ] **Testing notes** → Known issues or browser-specific quirks
- [ ] **Maintenance notes** → How to update content, add new sections

---

## Common Patterns from Flaneur 718 Project

### Chapter Transition Animation Pattern

```javascript
// Typewriter effect on scroll into view
const chapterObserver = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting && !entry.target.classList.contains('typed')) {
      const text = "CHAPTER 1";
      typewriterEffect(entry.target, text, 90);
      chapterObserver.unobserve(entry.target);
    }
  });
}, { threshold: 0.5 });
```

### Scrapbook Artifact Drop Pattern

```javascript
// Trigger after user reads paragraph
setTimeout(() => {
  artifacts.forEach((artifact, index) => {
    setTimeout(() => {
      artifact.classList.add('fl718-artifact--visible');
    }, index * 400); // 400ms stagger
  });
}, 3500); // Wait 3.5 seconds after section visible
```

### Film Strip Horizontal Scroll Pattern

```javascript
const filmStripWidth = filmStrip.scrollWidth;
const viewportWidth = window.innerWidth;
const maxScroll = filmStripWidth - viewportWidth;

gsap.to('#film-strip', {
  x: -maxScroll,
  ease: 'none',
  scrollTrigger: {
    trigger: '.film-strip-section',
    start: 'top top',
    end: '+=3000',
    scrub: 1,
    pin: true,
    anticipatePin: 1,
  }
});
```

---

## Troubleshooting Image Loading Issues

### Problem: Images Show as Gray Placeholders

**Symptoms:**
- Images appear as solid gray rectangles
- Captions display correctly
- Container elements are visible but images are not

**Common Causes & Solutions:**

#### 1. URL Encoding for File Paths (file:// protocol)

**Problem:** When opening HTML files directly in browser (file:// protocol), folder names with spaces cause image loading failures.

**Cause:** Browsers require URL encoding for special characters in file paths.

**Solution:** Encode spaces as `%20` in all image src attributes:

```html
<!-- ❌ Won't work with file:// protocol -->
<img src="2. chapter 2/nook-nook-1.png">

<!-- ✅ Works correctly -->
<img src="2.%20chapter%202/nook-nook-1.png">
```

**Find/Replace Pattern:**
```javascript
// Replace all folder paths with URL-encoded versions
src="0. Hero/        →  src="0.%20Hero/
src="1. Chapter 1/   →  src="1.%20Chapter%201/
src="2. chapter 2/   →  src="2.%20chapter%202/
```

**Note:** This is only needed for local file:// testing. WordPress Media Library URLs don't have this issue.

---

#### 2. CSS Class Structure Conflicts

**Problem:** Using `.fl718-photo__image` class directly on `<img>` tags causes blur filter to be applied.

**Cause:** The `.fl718-photo__image` class was designed as a **container div** for Polaroid development effects with blur filters, not for direct use on img tags.

**Wrong Structure:**
```html
<!-- ❌ Applies blur filter to image -->
<img src="photo.jpg" class="fl718-photo__image">
```

**Correct Structure for Polaroids:**
```html
<!-- ✅ Blur filter on container, img inside -->
<div class="fl718-photo__image">
  <img src="photo.jpg" alt="Description">
</div>
```

**Correct Structure for Product Photos:**
```html
<!-- ✅ No wrapper, or use different class -->
<img src="photo.jpg" alt="Description">
```

---

#### 3. Unwanted CSS Filters on Images

**Problem:** Images are blurry or appear overexposed/washed out.

**Cause:** CSS filters meant for one effect (Polaroid development) being applied to regular photos.

**Diagnosis:**
```css
/* Check for these filters in your CSS */
.fl718-photo__image {
  filter: brightness(3.5) contrast(0.05) saturate(0) blur(6px);
  /* ☝️ This makes images gray/blurry */
}
```

**Solution:** Override filters for specific contexts:
```css
/* Remove Polaroid effects from product photos */
.fl718-photo-grid .fl718-photo__image {
  filter: none !important;
  background: transparent;
}

.fl718-photo-grid .fl718-photo__image img {
  width: 100%;
  height: auto;
  display: block;
}
```

---

#### 4. Debugging Strategy for Image Issues

**Step-by-step troubleshooting:**

1. **Check Browser Console** (F12 → Console tab)
   - Look for 404 errors (file not found)
   - Check for CORS errors (cross-origin issues)

2. **Inspect Element** (Right-click image → Inspect)
   - Check computed `filter` property
   - Check `opacity` (should be 1, not 0)
   - Check `display` (should not be "none")
   - Verify `src` attribute path

3. **Compare Working vs Non-Working**
   - If other images load correctly, compare their HTML structure
   - Check for class differences
   - Look for CSS specificity conflicts

4. **Test File Path Directly**
   - Copy image src path from inspector
   - Paste into browser address bar
   - If image doesn't load, path is wrong

5. **Check CSS Filters**
   ```javascript
   // In browser console, check applied filters
   const img = document.querySelector('.fl718-photo__image');
   console.log(getComputedStyle(img).filter);
   ```

---

### Class Naming Best Practices

**Lesson Learned:** Don't reuse the same class name for different purposes.

**Bad Example:**
```css
/* Using same class for two different purposes causes conflicts */
.fl718-photo__image {
  /* Polaroid container with blur effect */
  filter: brightness(3.5) contrast(0.05) saturate(0) blur(6px);
}

/* Later... trying to use for product photos */
.fl718-photo__image {
  /* Conflicts with Polaroid styles */
}
```

**Good Example:**
```css
/* Separate classes for different purposes */
.fl718-polaroid__image {
  /* Polaroid-specific blur effect */
  filter: brightness(3.5) contrast(0.05) saturate(0) blur(6px);
}

.fl718-product__image {
  /* Clean product photo display */
  width: 100%;
  display: block;
}
```

**Or use context-specific overrides:**
```css
/* Base Polaroid style */
.fl718-photo__image {
  filter: brightness(3.5) contrast(0.05) saturate(0) blur(6px);
}

/* Override for product grid context */
.fl718-photo-grid .fl718-photo__image {
  filter: none !important;
}
```

---

### Multi-Section Class Name Conflicts and Style Bleeding

**Discovered:** February 15, 2026 (Chapter 4 Header Bug)

#### Problem: Generic Class Names Reused Across Sections

**Issue:** When the same class name (e.g., `.fl718-opening`) is used in multiple sections with different styling needs, base styles from the first definition can "bleed" into later sections, causing unexpected layout and styling issues.

**Real Example - Chapter 4 Header Bug:**

**What Went Wrong:**
- Hero section defined `.fl718-opening` with these properties:
  ```css
  .fl718-opening {
    background: #ffffff;     /* ← Created white box */
    width: 100vw;
    min-height: 100vh;
    display: flex;           /* ← Broke layout */
  }
  ```

- Chapter 4 tried to redefine it:
  ```css
  .fl718-chapter4 .fl718-opening {
    max-width: 1000px;
    margin: 0 auto;
    padding: 180px 60px 120px 60px;
    text-align: center;
    /* ❌ Didn't override base styles! */
  }
  ```

**Result:**
- ❌ White box appeared (inherited `background: #ffffff`)
- ❌ Text positioned incorrectly (inherited `display: flex`)
- ❌ Width issues (inherited `width: 100vw`)
- ❌ Chapter number hidden or misaligned

---

#### Diagnosis Steps

**Step 1: Compare Standalone vs Unified**
```bash
# Open standalone chapter file
open "4. chapter 4/about-me-chapter-4.html"

# Compare with unified version
# Look for visual differences in header/layout
```

**Step 2: Inspect Computed Styles**
```javascript
// In browser DevTools console:
const openingEl = document.querySelector('.fl718-chapter4 .fl718-opening');
console.log(getComputedStyle(openingEl).background);    // Check background
console.log(getComputedStyle(openingEl).display);       // Check display
console.log(getComputedStyle(openingEl).width);         // Check width
console.log(getComputedStyle(openingEl).minHeight);     // Check min-height
```

**Step 3: Check CSS Specificity**
```css
/* Lower specificity (Hero) */
.fl718-opening { }

/* Higher specificity (Chapter 4) - SHOULD override, but doesn't always! */
.fl718-chapter4 .fl718-opening { }

/* Problem: New properties are added, but old ones aren't overridden */
```

---

#### Solution: Explicit Property Overrides

**Pattern:** Always explicitly reset/override inherited properties from base class

```css
.fl718-chapter4 .fl718-opening {
  /* New styles specific to Chapter 4 */
  position: relative;
  max-width: 1000px;
  margin: 0 auto;
  padding: 180px 60px 120px 60px;
  text-align: center;

  /* ✅ CRITICAL: Override hero section base styles */
  background: transparent !important;   /* Reset background */
  width: auto !important;                /* Reset width */
  min-height: auto !important;           /* Reset min-height */
  display: block !important;             /* Reset display mode */
}
```

**Why `!important` is needed here:**
- CSS specificity is equal (both use class selectors)
- Source order matters - hero comes first, so it applies first
- Chapter 4 adds new properties but doesn't replace old ones
- `!important` forces the override

---

#### Prevention Strategies

**Strategy 1: Use Unique Class Names Per Section**

❌ **Bad - Generic Names:**
```css
/* Hero */
.fl718-opening { }

/* Chapter 4 */
.fl718-chapter4 .fl718-opening { }  /* ← Conflict risk! */
```

✅ **Good - Specific Names:**
```css
/* Hero */
.fl718-hero-opening { }

/* Chapter 4 */
.fl718-ch4-opening { }

/* No conflicts possible */
```

**Strategy 2: Establish Base → Override Pattern**

```css
/* Step 1: Define a true base class with minimal styles */
.fl718-section-base {
  position: relative;
  width: 100%;
  /* Only truly universal properties */
}

/* Step 2: Extend for each section */
.fl718-hero-section {
  /* Hero-specific */
  background: #ffffff;
  display: flex;
  min-height: 100vh;
}

.fl718-chapter4-section {
  /* Chapter 4-specific */
  background: #fef3e2;
  display: block;
  padding: 180px 60px;
}
```

**Strategy 3: Use CSS Custom Properties (Variables)**

```css
/* Define section-specific variables */
.fl718-hero-opening {
  --opening-bg: #ffffff;
  --opening-display: flex;
  --opening-width: 100vw;

  background: var(--opening-bg);
  display: var(--opening-display);
  width: var(--opening-width);
}

.fl718-chapter4 {
  /* Override variables at parent level */
  --opening-bg: transparent;
  --opening-display: block;
  --opening-width: auto;
}
```

---

#### Debugging Checklist for Similar Issues

When you see unexpected layout/styling in a section:

- [ ] **Check for reused class names** - Search CSS for all occurrences
- [ ] **Inspect computed styles** - Compare expected vs actual
- [ ] **Check CSS specificity** - Use DevTools to see which rule wins
- [ ] **Look for inherited properties** - `background`, `display`, `width`, `position`
- [ ] **Test standalone vs unified** - Does the section work in isolation?
- [ ] **Add explicit resets** - Override all potentially conflicting properties
- [ ] **Use `!important` strategically** - When specificity is equal and you need to force override
- [ ] **Consider renaming** - If conflicts are frequent, use unique class names

---

#### Common Properties That Cause Bleeding Issues

**Layout Properties:**
- `display` (flex, grid, block, inline-block)
- `width` / `max-width` / `min-width`
- `height` / `max-height` / `min-height`
- `position` (relative, absolute, fixed)

**Visual Properties:**
- `background` / `background-color`
- `border` / `border-radius`
- `box-shadow`
- `overflow` / `overflow-x` / `overflow-y`

**Spacing Properties:**
- `margin` / `padding`
- `gap` (for flex/grid)

**Text Properties:**
- `text-align`
- `font-size` / `font-weight` / `font-family`
- `color`
- `line-height`

---

#### Quick Reference: Override Template

```css
/* When you need to neutralize inherited styles from a base class */
.specific-context .reused-class {
  /* Reset display mode */
  display: block !important;           /* or: inline-block, flex, grid, none */

  /* Reset dimensions */
  width: auto !important;              /* or: 100%, specific px value */
  height: auto !important;
  min-width: 0 !important;
  min-height: 0 !important;
  max-width: none !important;
  max-height: none !important;

  /* Reset backgrounds */
  background: transparent !important;   /* or: specific color */

  /* Reset positioning */
  position: relative !important;        /* or: static, absolute */
  top: auto !important;
  left: auto !important;
  right: auto !important;
  bottom: auto !important;

  /* Reset spacing */
  margin: 0 !important;                /* or: specific values */
  padding: 0 !important;               /* or: specific values */

  /* Then add your specific styles */
  /* ... */
}
```

---

## Advanced Technical Patterns & Bug Fixes

### Animation Cleanup & Performance

#### Observer Cleanup Pattern

**Problem:** Animations re-trigger every time user scrolls back to section.

**Solution:** Always unobserve after animation completes to prevent re-triggering:

```javascript
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      // Trigger animation
      entry.target.classList.add('animate');

      // ✅ CRITICAL: Unobserve to prevent re-trigger
      observer.unobserve(entry.target);
    }
  });
}, { threshold: 0.3 });

observer.observe(element);
```

#### Event Listener Cleanup

**Problem:** Scroll listeners continue running after animation completes, wasting resources.

**Solution:** Remove event listeners after they're no longer needed:

```javascript
let animationTriggered = false;

function checkScrapbookTrigger() {
  if (shouldTrigger && !animationTriggered) {
    animateScrapbook();
    animationTriggered = true;

    // ✅ Clean up listener
    window.removeEventListener('scroll', checkScrapbookTrigger);
  }
}

window.addEventListener('scroll', checkScrapbookTrigger);
```

#### Will-Change Cleanup

**Problem:** `will-change` property uses extra memory if left on elements permanently.

**Solution:** Remove `will-change` after animation completes:

```javascript
// Set will-change before animation
element.style.willChange = 'transform, opacity';

// Animate
gsap.to(element, {
  opacity: 1,
  y: 0,
  duration: 1,
  onComplete: () => {
    // ✅ Clean up will-change after animation
    element.style.willChange = 'auto';
  }
});
```

Or set via CSS and cleanup after delay:

```javascript
setTimeout(() => {
  photos.forEach(photo => photo.style.willChange = 'auto');
}, 2500); // After all animations complete
```

---

### GSAP Advanced Patterns

#### Overwrite: 'auto' for Hover Conflicts

**Problem:** Hover animations conflict with ongoing scroll animations, causing jank.

**Solution:** Use `overwrite: 'auto'` to intelligently cancel conflicting animations:

```javascript
artifact.addEventListener('mouseenter', () => {
  gsap.to(artifact, {
    y: -15,
    scale: 1.05,
    boxShadow: '0 20px 40px rgba(0, 0, 0, 0.3)',
    duration: 0.35,
    ease: 'power2.out',
    overwrite: 'auto',  // ✅ Cancels only conflicting properties
  });
});

artifact.addEventListener('mouseleave', () => {
  gsap.to(artifact, {
    y: 0,
    scale: 1,
    boxShadow: '0 8px 24px rgba(0, 0, 0, 0.18)',
    duration: 0.5,
    ease: 'power2.out',
    overwrite: 'auto',  // ✅ Prevents animation stacking
  });
});
```

**Without `overwrite`:** Animations stack and conflict, causing stuttering.
**With `overwrite: 'auto'`:** Only conflicting properties are overwritten, smooth transitions.

#### gsap.set() for Initial States

**Problem:** Elements flash in default state before animation starts.

**Solution:** Use `gsap.set()` to immediately set initial state (no animation):

```javascript
const artifact = document.querySelector('.artifact');

// ✅ Set initial state instantly (no transition)
gsap.set(artifact, {
  opacity: 0,
  x: -280,
  y: -90,
  rotation: 20,
  scale: 1.06,
});

// Then animate from that state
gsap.to(artifact, {
  opacity: 1,
  x: 0,
  y: 0,
  rotation: -2,
  scale: 1,
  duration: 0.8,
  ease: 'power3.out',
});
```

#### gsap.delayedCall() for Post-Animation Adjustments

**Problem:** Need to make subtle adjustments after animation completes (realistic "hand" adjustments).

**Solution:** Use `gsap.delayedCall()` for timed callbacks:

```javascript
// Main animation completes
animateScrapbook();

// Realistic adjustments - person goes back to nudge items
gsap.delayedCall(3.8, () => {
  const postcards = document.querySelector('.artifact--postcards');
  if (postcards) {
    gsap.to(postcards, {
      x: '+=5',
      y: '-=3',
      rotation: '+=0.6',
      duration: 0.28,
      ease: 'power1.inOut'
    });
  }
});

gsap.delayedCall(4.2, () => {
  const mat = document.querySelector('.artifact--mat');
  if (mat) {
    gsap.to(mat, {
      x: '-=4',
      rotation: '-=0.7',
      duration: 0.22,
      ease: 'power1.inOut'
    });
  }
});
```

**Benefits:**
- More organized than nested `setTimeout`
- Automatically paused/resumed with GSAP globalTimeline
- Can be killed with `gsap.killTweensOf()`

#### Scrub Speed Optimization

**Problem:** Default `scrub: true` can feel too directly tied to scroll (no smoothing).

**Solution:** Use `scrub: 1` (1 second smoothing) or `scrub: 0.5` (faster) for polish:

```javascript
// Default - tied directly to scroll position
gsap.to('.element', {
  x: -1000,
  scrollTrigger: {
    trigger: '.section',
    start: 'top top',
    end: '+=3000',
    scrub: true,  // ❌ Can feel mechanical
    pin: true,
  }
});

// Better - smooth with 1s catch-up delay
gsap.to('.element', {
  x: -1000,
  scrollTrigger: {
    trigger: '.section',
    start: 'top top',
    end: '+=3000',
    scrub: 1,  // ✅ Smooth, polished feel
    pin: true,
  }
});

// Fastest - for quicker animations
gsap.to('.thesis-text', {
  filter: 'brightness(1) contrast(1)',
  scrollTrigger: {
    trigger: '.thesis-section',
    start: 'top 70%',
    end: 'center 30%',
    scrub: 0.5,  // ✅ Faster response for text effects
  }
});
```

**Rule of thumb:**
- `scrub: 1` - Most animations (smooth, polished)
- `scrub: 0.5` - Quick text/color transitions
- `scrub: 2` - Very smooth but can feel sluggish

---

### CSS Layout Techniques

#### Pointer-Events: None for Overlay Elements

**Problem:** Overlay elements (like Polaroid development overlays) block clicks on images beneath.

**Solution:** Use `pointer-events: none` to allow clicks through:

```css
.fl718-photo__overlay {
  position: absolute;
  top: 12px;
  left: 12px;
  right: 12px;
  bottom: 42px;
  background: rgba(255, 255, 255, 0.85);
  opacity: 1;
  z-index: 5;
  pointer-events: none;  /* ✅ Allow clicks through to image */
}
```

**Use cases:**
- Polaroid development overlays
- Film strip edge codes
- Hand-drawn selection marks
- Decorative SVG elements
- Loading indicators

#### Z-Index Management Strategy

**Problem:** Elements overlap incorrectly, especially during animations.

**Solution:** Use CSS custom properties for dynamic z-index management:

```css
/* Define z-index hierarchy */
.fl718-artifact--library { z-index: 4; }
.fl718-artifact--metrocard { z-index: 6; }
.fl718-artifact--spraypaint { z-index: 3; }
.fl718-artifact--drinks { z-index: 5; }
.fl718-artifact--pizza { z-index: 2; }
.fl718-artifact--notes { z-index: 7; }
```

```javascript
// On hover, bring to front temporarily
artifact.addEventListener('mouseenter', () => {
  artifact.style.zIndex = '100';  // ✅ Bring to top
  // ... animation
});

artifact.addEventListener('mouseleave', () => {
  // ... animation
  setTimeout(() => {
    artifact.style.zIndex = '';  // ✅ Return to original z-index
  }, 500); // After animation completes
});
```

---

### Accessibility Best Practices

#### Comprehensive Reduced Motion Support

**Problem:** Animations can cause motion sickness or be distracting for some users.

**Solution:** Implement comprehensive `prefers-reduced-motion` checks:

```css
/* CSS - Disable all animations */
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

```javascript
// JavaScript - Skip complex animations
const prefersReducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches;

function animateScrapbook() {
  const artifacts = document.querySelectorAll('.fl718-artifact');

  if (prefersReducedMotion) {
    // ✅ Instant reveal - no animation
    artifacts.forEach(artifact => {
      artifact.classList.add('fl718-artifact--visible');
    });
    return;
  }

  // Regular animation for users who prefer motion
  artifacts.forEach((artifact, index) => {
    setTimeout(() => {
      artifact.classList.add('fl718-artifact--visible');
    }, index * 400);
  });
}
```

**Implementation checklist:**
- ✅ CSS: Global animation disable
- ✅ JS: Check `prefers-reduced-motion` before complex animations
- ✅ Fallback: Instant reveal with final state
- ✅ GSAP: Skip ScrollTrigger animations on reduced motion preference

---

### File System & Deployment Issues

#### Folder Name Casing Inconsistency

**Problem Discovered:** Project has inconsistent folder naming casing:
- `1. Chapter 1` (capital C)
- `2. chapter 2`, `3. chapter 3`, `4. chapter 4`, `5. chapter 5` (lowercase c)

**Risk:** On case-sensitive file systems (Linux servers, some CDNs), this can cause:
- Broken image links
- 404 errors in production
- Works locally (macOS/Windows are case-insensitive) but fails on server

**Solution:**

**Option 1: Standardize folder names (Recommended)**
```bash
# Rename all to consistent lowercase
mv "1. Chapter 1" "1. chapter 1"
mv "2. chapter 2" "2. chapter 2"  # Already lowercase
# ... etc
```

**Option 2: Update all references in code**
```bash
# Find and replace in HTML
# "1. Chapter 1/" → "1. chapter 1/"
# "0. Hero/" → "0. hero/"
```

**Prevention:**
- Always use lowercase for folders and files
- Use hyphens, not spaces: `chapter-1/` not `Chapter 1/`
- Test on case-sensitive filesystem before deployment

#### URL Encoding for File:// Protocol

**Already documented** - see "Troubleshooting Image Loading Issues" section above.

---

### Animation Timing & Choreography

#### Staggered Entry Timing Pattern

**Pattern Used:** 3.5 second delay before scrapbook items drop, giving user time to read paragraph.

```javascript
// User reads paragraph (natural pause)
setTimeout(() => {
  artifacts.forEach((artifact, index) => {
    setTimeout(() => {
      artifact.classList.add('fl718-artifact--visible');
    }, index * 400); // 400ms stagger between items
  });
}, 3500); // ✅ 3.5s initial delay for reading
```

**Timing Guidelines:**
- **Initial delay:** 3-3.5s (paragraph reading time)
- **Stagger between items:** 300-400ms (readable sequence)
- **Polaroid development:** 4s (realistic chemistry timing)
- **Typewriter speed:** 50-80ms per char, 200ms pause on punctuation

#### Variable Speed Typewriter

**Pattern:** Realistic typing with variable speed and punctuation pauses.

```javascript
async function typewriterEffect3() {
  for (let i = 0; i < fullText.length; i++) {
    typewriterSentence.textContent = fullText.substring(0, i + 1);

    // ✅ Variable speed - faster for letters, slower for punctuation
    const speed = fullText[i] === ',' || fullText[i] === '.'
      ? 200  // Pause on punctuation
      : 50 + Math.random() * 30;  // Variable speed 50-80ms

    await new Promise(resolve => setTimeout(resolve, speed));
  }
}
```

---

## Section Transitions & Film Perforation Bar

**Discovered:** February 15, 2026 (Hero → Chapter 1 Transition)

### Use Case: Marking Chapter Transitions

When creating multi-section biographical or narrative pages, you may need visual transitions between major sections that:
- Mark the shift from one narrative moment to another
- Are minimal but intentional
- Match the emotional/thematic arc
- Continue the visual language

### Film Perforation Bar Pattern

**Concept:** A thin horizontal black strip with film sprocket holes on the edges and Kodak-style edge code text in the center.

**Use Case:** Perfect for photography-focused sites transitioning between sections, especially when moving from "present-day photographer" to "beginning of chronological story."

**Visual Elements:**
- Black background strip (50-60px height)
- White sprocket holes on left and right edges (mimics 35mm film)
- Orange Kodak-style edge code text in center (e.g., "KODAK TRI-X 400")
- Minimal, geometric, authentic to film photography

### Implementation Code

**HTML:**
```html
<!-- Film Perforation Bar Transition -->
<div class="fl718-transition-perforation">
  <!-- Left perforations -->
  <div class="fl718-perforation-left">
    <div class="fl718-perf-row">
      <div class="fl718-perforation-hole"></div>
      <div class="fl718-perforation-hole"></div>
    </div>
    <div class="fl718-perf-row">
      <div class="fl718-perforation-hole"></div>
      <div class="fl718-perforation-hole"></div>
    </div>
    <div class="fl718-perf-row">
      <div class="fl718-perforation-hole"></div>
      <div class="fl718-perforation-hole"></div>
    </div>
  </div>

  <!-- Right perforations -->
  <div class="fl718-perforation-right">
    <div class="fl718-perf-row">
      <div class="fl718-perforation-hole"></div>
      <div class="fl718-perforation-hole"></div>
    </div>
    <div class="fl718-perf-row">
      <div class="fl718-perforation-hole"></div>
      <div class="fl718-perforation-hole"></div>
    </div>
    <div class="fl718-perf-row">
      <div class="fl718-perforation-hole"></div>
      <div class="fl718-perforation-hole"></div>
    </div>
  </div>

  <!-- Edge code text -->
  <div class="fl718-transition-film-code">Kodak Tri-X 400</div>
</div>
```

**CSS:**
```css
/* Container */
.fl718-transition-perforation {
  position: relative;
  width: 100%;
  background: #000000;
  height: 60px;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: visible;
}

/* Left perforations */
.fl718-perforation-left {
  position: absolute;
  left: 20px;
  top: 0;
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: space-evenly;
  padding: 10px 0;
  gap: 4px;
}

/* Right perforations */
.fl718-perforation-right {
  position: absolute;
  right: 20px;
  top: 0;
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: space-evenly;
  padding: 10px 0;
  gap: 4px;
}

.fl718-perf-row {
  display: flex;
  gap: 8px;
}

.fl718-perforation-hole {
  width: 7px;
  height: 7px;
  background: #ffffff;
  border-radius: 1px;
}

/* Edge code text - UNIQUE CLASS NAME CRITICAL */
.fl718-transition-film-code {
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  font-family: 'Courier New', monospace;
  font-size: 12px;
  font-weight: 700;
  color: #ff6600;
  letter-spacing: 4px;
  text-transform: uppercase;
  white-space: nowrap;
  display: inline-block;
}

/* Mobile */
@media (max-width: 768px) {
  .fl718-transition-perforation {
    height: 40px;
  }

  .fl718-perforation-hole {
    width: 6px;
    height: 6px;
  }

  .fl718-transition-film-code {
    font-size: 10px;
    letter-spacing: 2px;
  }
}
```

---

### CRITICAL BUG: CSS Class Name Conflicts Across Sections

**Discovered:** February 15, 2026 (Film Perforation Bar Text Bug)

#### The Problem

When implementing the film perforation bar transition, the edge code text ("KODAK TRI-X 400") refused to display horizontally despite correct CSS. Multiple attempts to fix orientation failed silently.

**Symptoms:**
- Text displayed vertically (rotated 90°) instead of horizontally
- Text was 8px instead of intended 12px
- CSS changes had NO effect when refreshing browser
- `writing-mode: horizontal-tb` was ignored
- `display: inline-block` was ignored
- Font size changes were ignored

**Root Cause:**
```css
/* TRANSITION BAR (Line ~475) */
.fl718-film-edge-code {
  font-size: 12px;
  /* ... horizontal display styles ... */
}

/* CHAPTER 4 FILM STRIP (Line ~1586) - COMES LATER */
.fl718-film-edge-code {
  font-size: 8px;
  writing-mode: vertical-rl;
  transform: rotate(180deg);
  /* ... vertical display styles for Chapter 4 ... */
}
```

**What happened:**
1. Two different sections used the SAME class name: `.fl718-film-edge-code`
2. Chapter 4's CSS definition came AFTER the transition bar's CSS
3. CSS cascade rules: later styles override earlier ones with same specificity
4. Chapter 4's vertical text styles completely overrode transition bar's horizontal styles
5. ALL transition bar styling attempts were being silently overwritten

#### Debugging Process

**Step 1: Verify Changes Saved**
```bash
# Read the CSS file to confirm changes were written
# Result: ✅ Changes were saved correctly
```

**Step 2: Check for Global Writing-Mode Rules**
```bash
grep "writing-mode" about-me-unified-prototype.html
```

**Result:** Found DUPLICATE class name at line 1586:
```css
.fl718-film-edge-code {
  writing-mode: vertical-rl;  /* ← This was overriding everything! */
  transform: rotate(180deg);
  font-size: 8px;
}
```

**Step 3: Identify Conflict**
- Transition bar used `.fl718-film-edge-code` (line ~475)
- Chapter 4 film strip ALSO used `.fl718-film-edge-code` (line ~1586)
- Chapter 4's definition comes 1100+ lines LATER
- CSS cascade made Chapter 4's styles win

#### The Solution

**Create unique class names for each section's edge code elements:**

```css
/* ✅ GOOD: Transition bar has unique class */
.fl718-transition-film-code {
  font-size: 12px;
  display: inline-block;
  /* ... horizontal styles ... */
}

/* ✅ GOOD: Chapter 4 has its own class (unchanged) */
.fl718-film-edge-code {
  writing-mode: vertical-rl;
  transform: rotate(180deg);
  /* ... vertical styles for Chapter 4 film strip ... */
}
```

**Updates Required:**
1. **CSS:** Rename `.fl718-film-edge-code` → `.fl718-transition-film-code` (line ~476)
2. **HTML:** Update class in transition bar div (line ~2045)
3. **Mobile CSS:** Update media query selector (line ~502)

**Result:** ✅ Text immediately displayed horizontally as intended

---

### Class Naming Best Practices for Multi-Section Pages

#### The Problem with Generic Names

❌ **BAD - Reusing Generic Class Names:**
```css
/* Hero Section */
.fl718-edge-code { /* ... */ }

/* Transition Bar */
.fl718-edge-code { /* ... */ }  /* ← CONFLICT! */

/* Chapter 4 */
.fl718-edge-code { /* ... */ }  /* ← OVERWRITES BOTH! */
```

**Result:** Whichever definition comes LAST in the CSS file will override all others.

#### Solution Patterns

**Pattern 1: Section-Specific Prefixes**
```css
/* ✅ GOOD: Each section has unique prefix */
.fl718-hero-edge-code { /* ... */ }
.fl718-transition-film-code { /* ... */ }
.fl718-chapter4-edge-code { /* ... */ }
```

**Pattern 2: BEM with Section Context**
```css
/* ✅ GOOD: Block__Element naming with section */
.fl718-transition__film-code { /* ... */ }
.fl718-chapter4__film-code { /* ... */ }
```

**Pattern 3: Unique Descriptive Names**
```css
/* ✅ GOOD: Describes purpose AND location */
.fl718-perforation-bar-text { /* ... */ }
.fl718-horizontal-film-strip-code { /* ... */ }
```

#### Debugging Checklist: "My CSS Changes Don't Work"

When CSS changes appear to have no effect:

**1. Verify Changes Saved**
```bash
# Read the file to confirm
cat yourfile.html | grep "your-class-name" -A 5
```

**2. Check for Duplicate Class Names**
```bash
# Search for all instances of the class
grep "\.your-class-name" yourfile.html -n

# Count how many times it appears
grep "\.your-class-name" yourfile.html | wc -l
```

**3. Check CSS Cascade Order**
- Which definition comes first?
- Which comes last? (Last one usually wins)
- Are specificity levels equal?

**4. Check Browser DevTools**
- Right-click element → Inspect
- Look at "Styles" panel
- See which rules are crossed out (overridden)
- Check which file/line is applying the final style

**5. Clear Browser Cache**
```bash
# Hard refresh
# Chrome/Firefox: Cmd+Shift+R (Mac) or Ctrl+Shift+R (Windows)
# Safari: Cmd+Option+R
```

**6. Check for !important Overrides**
```css
/* Later rules with !important will win */
.class { color: red; }
.class { color: blue !important; }  /* ← This wins */
```

**7. Verify Selector Specificity**
```css
/* Higher specificity wins */
.class { }                    /* Specificity: 10 */
div.class { }                 /* Specificity: 11 */
#id .class { }                /* Specificity: 110 */
```

#### Prevention Strategy

**Rule:** Every reusable component in different sections should have a unique class name.

**Naming Convention:**
```
.fl718-[section]-[element]-[variant]
       ↓        ↓         ↓
    prefix   location  description

Examples:
.fl718-transition-film-code
.fl718-hero-title-main
.fl718-chapter1-artifact
.fl718-chapter4-film-edge-code
```

**Avoid:**
- Generic names like `.edge-code`, `.title`, `.text`
- Reusing the same class across multiple sections
- Assuming CSS order doesn't matter (it does!)

---

## Version History

**v2.4 - February 15, 2026**
- ✨ **NEW:** Added "Section Transitions & Film Perforation Bar" section
  - Complete implementation guide for film perforation bar transition
  - HTML/CSS code for authentic 35mm film aesthetic
  - Mobile responsive patterns
- 🐛 **DISCOVERED BUG #3:** CSS class name conflicts across sections
  - **Problem:** `.fl718-film-edge-code` used in both transition bar AND Chapter 4
  - **Symptom:** CSS changes had no effect, text stayed vertical despite horizontal CSS
  - **Root Cause:** Chapter 4's CSS (line 1586) came AFTER transition CSS (line 476), overriding all styles via cascade
  - **Solution:** Created unique class `.fl718-transition-film-code` for transition bar only
  - **Debugging Process:** Documented 7-step checklist for "CSS changes don't work" scenarios
  - **Prevention:** Section-specific class naming convention documented
- 📚 **NEW:** "Class Naming Best Practices for Multi-Section Pages"
  - Pattern examples: section-specific prefixes, BEM with context, descriptive names
  - Anti-patterns: generic reused names, assuming CSS order doesn't matter
  - Rule: Every section's components need unique class names
- 🔍 **ENHANCED:** Debugging workflows
  - grep patterns for finding duplicate class names
  - CSS cascade order checking
  - Browser DevTools inspection workflow
  - Specificity calculation examples

**v2.3 - February 15, 2026**
- ✨ **NEW:** Added "Multi-Section Class Name Conflicts and Style Bleeding" section
- ✨ **NEW:** Added "Common IntersectionObserver Issues & Fixes" section
- **DISCOVERED BUG #1:** Chapter 4 header - white box and layout issues caused by reused `.fl718-opening` class
  - Documented how base styles from one section bleed into another when generic class names are reused
  - Added diagnosis steps: compare standalone vs unified, inspect computed styles, check specificity
  - Solution pattern: Explicit property overrides with `!important` when needed
  - Prevention strategies: Use unique class names, establish base → override pattern, CSS custom properties
- **DISCOVERED BUG #2:** Chapter 4 typewriter - animation never triggered (cursor blinked but no text)
  - Problem 1: DOMContentLoaded wrapper when script at end of body (observer never set up)
  - Problem 2: Strict `textContent === ''` check fails with whitespace
  - Problem 3: Threshold too high (0.2) misses fast scrolling
  - Problem 4: No re-trigger prevention causes multiple animations
  - Problem 5: rootMargin confusion (negative = later, positive = earlier)
  - Complete IntersectionObserver setup template with all best practices
  - Debugging checklist with console logging patterns
- **REFINED:** Chapter 4 typewriter timing - iterative optimization process
  - Problem 6: Finding the right animation trigger timing through user feedback
  - Documented iterative refinement process: conservative → aggressive → sweet spot
  - Added rootMargin value guidelines table (50px to 500px+ with feel descriptions)
  - Pro tips for timing tuning: test on actual device, remove delays during testing, consider scroll speed
  - Real example: -100px (too slow) → 300px (too fast) → 150px (perfect)
  - Testing checklist for different scroll speeds and UX feel
  - Emphasized timing as UX work requiring subjective user feedback
- Debugging checklist for similar layout/styling conflicts
- Common properties that cause bleeding issues (display, width, background, etc.)
- Quick reference override template for neutralizing inherited styles
- Real-world examples with before/after code from actual project bugs

**v2.2 - February 14, 2026**
- ✨ **NEW:** Added comprehensive "Advanced Technical Patterns & Bug Fixes" section
- Observer cleanup patterns (unobserve to prevent re-triggering)
- Event listener cleanup (removeEventListener for performance)
- GSAP `overwrite: 'auto'` for hover animation conflicts
- GSAP `gsap.set()`, `gsap.delayedCall()`, and `scrub` speed optimization
- `pointer-events: none` for overlay elements
- Z-index management strategies
- Comprehensive reduced motion accessibility implementation
- **DISCOVERED BUG:** Folder name casing inconsistency (Chapter 1 vs chapter 2)
- Animation timing & choreography best practices
- Variable speed typewriter implementation
- Will-change cleanup patterns

**v2.1 - February 14, 2026**
- Added comprehensive "Troubleshooting Image Loading Issues" section
- Documented URL encoding requirements for file:// protocol
- Added CSS class structure conflict resolution patterns
- Added image debugging step-by-step workflow
- Documented class naming best practices for avoiding conflicts

**v2.0 - February 2026**
- Consolidated all technical learnings from Flaneur 718 project
- Added GSAP ScrollTrigger integration
- Added IntersectionObserver patterns
- Added film photography aesthetics
- Added scrapbook animation patterns
- Added mobile optimization strategies
- Added debugging workflows
- Added performance best practices

**v1.0 - Initial Version**
- Basic WordPress/Uncode integration
- Overflow fixes
- Double scrollbar solutions

---

**End of Technical Reference**

*This document should be updated as new patterns and solutions are discovered. Each project teaches new techniques - document them here for future reference.*
