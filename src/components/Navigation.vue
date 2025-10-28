<template>
  <nav class="navigation">
    <div class="navigation__container">
      <router-link to="/" class="navigation__link" active-class="navigation__link--active">
        3D Apple Demo
      </router-link>
      <router-link to="/depth-demo" class="navigation__link" active-class="navigation__link--active">
        Depth Map Parallax
      </router-link>
    </div>
  </nav>
</template>

<script setup>
</script>

<style lang="scss" scoped>
.navigation {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 1000;
  background: white;
  padding: 1rem 2rem;

  &__container {
    display: flex;
    gap: 2rem;
    max-width: 1200px;
    margin: 0 auto;
  }

  &__link {
    color: black;
    text-decoration: none;
    font-weight: 600;
    font-size: 1rem;
    padding: 0.5rem 1rem;
    border-radius: 8px;
    transition: all 0.3s ease;
    background: lightblue;

    &:hover {
      background: rgba(255, 255, 255, 0.2);
      transform: translateY(-2px);
    }

    &--active {
      background: lightblue;
      box-shadow: 0 4px 15px rgba(79, 195, 247, 0.3);
      border: 1px solid slateblue;
    }
  }
}
</style>

