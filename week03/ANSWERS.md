## 1. In EventList, what goes wrong if you use the array index as `:key` and the list gets filtered while a card has local state (for example an open details panel)?
 
When the list is filtered, the cards shift position, so index 0 now belongs to a different event than before. Vue matches elements by key and reuses the existing component instance for the same key, so it keeps the old card's local state and only patches in the new props. The result is that an open details panel stays at the same position but now shows on the wrong event. With `:key="e.id"` the key follows the event, so each card keeps its own state.
 
## 2. Why is `visible` a `computed` and not a function called from the template? What is the difference when you type in the search box?
 
A `computed` is cached and only recalculates when something it depends on changes. A function called from the template runs again on every re-render of the component, whatever caused it. The template reads `visible` three times (the count, the `v-if` and the `v-for`), so a function would run the filter three times per render, while the computed runs it once and reuses the result. When typing in the search box, `search` changes on every keystroke, so both versions recalculate. The difference shows up when the component re-renders for an unrelated reason: the computed returns its cached value, while the function filters again.
 
## 3. EventCard emits `book` instead of changing `event.booked`. Give two reasons.
 
First, props are read-only, so a child that changes data it does not own breaks one-way data flow, and Vue warns about it. Second, the parent owns the data and the rules: `App` checks that a seat is free and updates `myBookings`, and a card that changed `booked` itself would skip both. Emitting also keeps `EventCard` reusable and easy to test, because it only reports "the user wants to book event 7".
 
## 4. You write `const { booked } = reactive(event); booked++`. The page does not change. Why?
 
Destructuring copies the number into a plain variable, so `booked` is no longer connected to the reactive object. Changing it changes only that local copy, `event.booked` is untouched, and Vue has nothing to track, so the page does not update. Also, because `booked` is declared with `const`, `booked++` would actually throw a `TypeError`.
 
## 5. Where does `myBookings` live, and why not in EventCard or EventList?
 
It lives in `App.vue`, next to `events` and `onBook`, because `App` is the one component that owns the data and changes it. The header total is shown in `App` and counts bookings across all events, so it needs the whole list. `EventCard` knows about one event only, and `EventList` is a poor place too, because the header is outside it and `App` could not read the list's state without extra events. A card that is filtered out is also unmounted, so its state would be lost.