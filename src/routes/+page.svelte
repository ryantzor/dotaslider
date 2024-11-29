<div
  class="carouselContainer"
  on:mousedown={handleDragStart}
  on:mousemove={handleDragMove}
  on:mouseup={handleDragEnd}
  on:mouseleave={handleDragEnd}
  on:touchstart={handleDragStart}
  on:touchmove={handleDragMove}
  on:touchend={handleDragEnd}
>
  <div class="carouselOfImages">
    {#each items as { name, imageUrl }}
      <div
        class="carouselImage"
        style="background-image: url({imageUrl});"
        title={name}
      >
        <div class="imageOverlay">{name}</div>
      </div>
    {/each}
  </div>
</div>



<script>
  import { onMount } from "svelte";
  import data from "/src/dota2_heroes.json"; // Import the JSON file

  let carousel;
  let selectedIndex = 0;
  let items = []; // Store the JSON data here
  let isDragging = false;
  let startX = 0;
  let currentOffset = 0;

  const settings = {
    autoPlay: false,
    wrapAround: true,
    visibleCells: 3,
    cellWidth: 0.2, // 20% of the container width
  };

  const resizeCells = () => {
    const totalCells = items.length;

    items.forEach((_, index) => {
      const cell = document.querySelector(`.carouselImage:nth-child(${index + 1})`);
      if (!cell) return;

      cell.classList.remove(
        "is-selected",
        "nextToSelectedLeft",
        "nextToSelectedRight",
        "nextToSelectedLeft2",
        "nextToSelectedRight2"
      );
      if (index === selectedIndex) {
        cell.classList.add("is-selected");
      } else if (index === (selectedIndex - 1 + totalCells) % totalCells) {
        cell.classList.add("nextToSelectedLeft");
      } else if (index === (selectedIndex - 2 + totalCells) % totalCells) {
        cell.classList.add("nextToSelectedLeft2");
      } else if (index === (selectedIndex + 1) % totalCells) {
        cell.classList.add("nextToSelectedRight");
      } else if (index === (selectedIndex + 2) % totalCells) {
        cell.classList.add("nextToSelectedRight2");
      }
    });
  };

  const moveCarousel = (direction) => {
    const totalCells = items.length;
    if (settings.wrapAround) {
      selectedIndex = (selectedIndex + direction + totalCells) % totalCells;
    } else {
      selectedIndex = Math.max(
        0,
        Math.min(selectedIndex + direction, totalCells - 1)
      );
    }
    resizeCells();
  };

  const handleDragStart = (event) => {
    isDragging = true;
    startX = event.clientX || event.touches?.[0]?.clientX;
    currentOffset = 0;
    carousel.style.transition = "none"; // Disable smooth transitions while dragging
  };

  const handleDragMove = (event) => {
    if (!isDragging) return;
    const currentX = event.clientX || event.touches?.[0]?.clientX;
    currentOffset = currentX - startX;
    carousel.style.transform = `translateX(${currentOffset}px)`; // Move the carousel
  };

  const handleDragEnd = () => {
    if (!isDragging) return;
    isDragging = false;
    carousel.style.transition = "transform 0.5s ease"; // Re-enable smooth transitions
    if (Math.abs(currentOffset) > 50) {
      moveCarousel(currentOffset < 0 ? 1 : -1);
    } else {
      carousel.style.transform = ""; // Snap back
    }
  };

  onMount(() => {
    items = data; // Load the JSON data
    carousel = document.querySelector(".carouselOfImages");
    resizeCells();

    if (settings.autoPlay) {
      setInterval(() => moveCarousel(1), 3000);
    }
  });
</script>


<style>
  .carouselContainer {
    overflow: hidden;
    position: relative;
    width: 100%;
  }

  .carouselOfImages {
    display: flex;
    transition: transform 0.5s ease;
  }

  .carouselImage {
    flex: 0 0 20%;
    height: 200px;
    background-size: cover;
    background-position: center;
    border: 1.5px solid black;
    margin-right: 16px;
    text-align: center;
    position: relative;
    color: white;
  }

  .carouselImage .imageOverlay {
    position: absolute;
    bottom: 0;
    background: rgba(0, 0, 0, 0.5);
    width: 100%;
    padding: 8px;
    box-sizing: border-box;
    text-align: center;
    font-size: 14px;
  }
</style>
