- media query
```css
<style>
  .large-screens {
    display: none;
  }

  @media (min-width: 300px) {
    .large-screens {
      display: block;
    }
    .small-screens {
      display: none;
    }
  }
</style>

<div class="large-screens">
  I only show up on large screens.
</div>
<div class="small-screens">
  Meanwhile, you'll only see me on small ones.
</div>
```

