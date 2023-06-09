# "chat app like" layout demo

Fixed top header, flexible scrollable content, fixed bottom footer with input field.

## Problem #1 - constant viewport height

Visual viewport can change dynamically while browsing (e.g. browsers might minimize the address
bar). There is a [`dvh` size unit](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Values_and_units) to address that, which seems to be supported and works
as expected, except for the problem #2 below (the open keyboard on iOS).

### Solution

Create custom css var and update it via js on `visualViewport` change.
Originally taken from [here](https://css-tricks.com/the-trick-to-viewport-units-on-mobile/).
Slightly adjusted.

The core of the idea:

```css
:root {
	--vh: 1dvh;
}

.vh100 {
	height: calc(var(--vh, 1dvh) * 100);
}
```

```javascript
// shortened, see index.html for more
window.visualViewport.addEventListener('resize', () => {
	document.documentElement.style.setProperty(
		'--vh',
		`${window.visualViewport.height * 0.01}px`
	);
});
```

## iOS Problem #2 - the scrolled viewport with opened virtual keyboard

The problem is only related to iOS (mainly safari, but others browsers on iOS seem to mimick
the safari behavior):

- the opened kbd is always rendered "above" the viewport (the `interactive-widget=resizes-visual` meta viewport instruction seem not to be supported at the time of writing)
- if iOS thinks the focused input is too close to the the viewport bottom it scrolls the window down regardless of any fixed layout elements

On Androids the `interactive-widget=resizes-visual` + the `--vh` trick combo seem to work fine.

### The solution attempt

Scroll back to the top immediately. Unfortunately this is far from perfect. See [source](index.html) for more.

The core idea:

```javascript
// shortened
// ...from inside visualViewport change handler...
if (iOS() && !isZoomed() && _previousHeight - height >= KBD_HEIGHT) {
	window.dispatchEvent(new Event('ios_keyboard_maybe_open'));
}
// ...

// and later
window.addEventListener('ios_keyboard_maybe_open', () => {
	window.scroll({ top: 0, behavior: 'instant' });
});
```
