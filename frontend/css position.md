## static position
  - is the default positioning like in html, you write it, it can be by default inline element, or it could be on the next line.  
## relative position
  - static position with options to move relative to the static position.
  - It worth to mension that relative position just takes an element from the layout, but it's default (static position) place stays in document flow.
## absolute position
  - absolutely positions element inside of parent, and it comletele deletes the element from the document flow.
  - **IMPORTANT:** in order to position something relatively or absolutely to (inside of) parent, parent must **_NOT_** be positioned statically
## fixed position
  - it's like absolute position, but _it ignores all parents (based on entire HTML page)_.
  - and it _fixed to the screen_ even when scrolling (example would be a static top bar).
## sticky position
  - combination of relative and fixed, when scrolling, it comes to this position and then fixates on the screen (sticks) and scrolls along.