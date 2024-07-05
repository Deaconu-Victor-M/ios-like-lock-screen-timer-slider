<script>
  import { onMount } from "svelte";
  import { gsap } from "gsap";

  let isSlider = false;
  let sliderValue = 0;
  let buttonElement;
  let sliderElement;
  let showTime;
  let overlay;
  let startY;
  let initialSliderHeight;
  let initialSliderWidth;
  let startButton;
  let icon;

  const totalSegments = 10;
  let segments = Array(totalSegments).fill(0);

  function handlePointerDown(event) {
    event.stopPropagation();
    if (isSlider) return; // Do nothing if slider is already active
    isSlider = true;
    startY = event.clientY || event.touches[0].clientY;
    sliderValue = 0.25; // Set slider to default 30 minutes
    updateSegments();

    gsap.to(buttonElement, {
      duration: 0.2,
      width: "150px",
      height: "50vh",
      borderRadius: "55px",
      bottom: "27vh",
      right: "50%",
      x: "50%",
      backgroundColor: "#00000000",
      ease: "power2.out",
      onComplete: () => {
        initialSliderHeight = buttonElement.offsetHeight;
        initialSliderWidth = buttonElement.offsetWidth;
        gsap.set(overlay, { opacity: "100" });
        gsap.set(sliderElement, { display: "flex" });
        gsap.set(startButton, { display: "block" });
        gsap.set(showTime, { display: "block" });
        gsap.set(icon, { display: "none" });
      },
    });

    window.addEventListener("pointermove", handlePointerMove);
    window.addEventListener("pointerup", handlePointerUp);
    window.addEventListener("pointerdown", handlePointerDownOutside);
  }

  function handlePointerUp(event) {
    isSlider = false;
    window.removeEventListener("pointermove", handlePointerMove);
    window.removeEventListener("pointerup", handlePointerUp);
  }

  function handlePointerDownOutside(event) {
    if (!buttonElement.contains(event.target)) {
      gsap.to(buttonElement, {
        duration: 0.3,
        width: "60px",
        height: "60px",
        borderRadius: "50%",
        bottom: "1rem",
        right: "1rem",
        x: 0,
        y: 0,
        ease: "power2.in",
        backgroundColor: "#0000007A",
        onStart: () => {
          gsap.set(sliderElement, { display: "none" });
          gsap.set(startButton, { display: "none" });
          gsap.set(showTime, { display: "none" });
          gsap.set(overlay, { opacity: "0" });
          gsap.set(icon, { display: "block" });
        },
      });
      window.removeEventListener("pointerdown", handlePointerDownOutside);
    }
  }

  function playStretchAnimation(deltaY) {
    const maxStretchHeight = 450 * 2; // Maximum stretch height in pixels
    const stretchRatio = Math.min(1, Math.abs(deltaY) / 200); // Adjust this ratio based on the desired sensitivity

    const newHeight =
      initialSliderHeight +
      (maxStretchHeight - initialSliderHeight) * stretchRatio;
    const newWidth = 130 * (1 - 0.8 * stretchRatio);

    gsap.to(buttonElement, {
      height: newHeight,
      width: newWidth,
      duration: 0.3,
      ease: "power1.out", // Changed to power1.out for smooth stretching
    });
  }

  function handlePointerMove(event) {
    event.preventDefault();
    if (isSlider) {
      const currentY = event.clientY || event.touches[0].clientY;
      const deltaY = startY - currentY;
      const sliderHeight = initialSliderHeight;
      let newValue = Math.max(
        0,
        Math.min(1, sliderValue + deltaY / sliderHeight)
      );

      const filledSegments = Math.floor(newValue * totalSegments);
      segments = segments.map((_, index) => (index < filledSegments ? 1 : 0));

      if (filledSegments === totalSegments || newValue === 0) {
        playStretchAnimation(deltaY);
      } else {
        // Reset to initial size dynamically
        gsap.to(buttonElement, {
          height: initialSliderHeight,
          width: initialSliderWidth,
          duration: 0.1,
          ease: "power2.out",
        });
      }

      sliderValue = newValue;
      startY = currentY;
    }
  }

  function updateSegments() {
    const filledSegments = Math.floor(sliderValue * totalSegments);
    segments = segments.map((_, index) => (index < filledSegments ? 1 : 0));
  }

  onMount(() => {
    gsap.set(sliderElement, { display: "none" });
    gsap.set(startButton, { display: "none" });
    gsap.set(showTime, { display: "none" });
    gsap.set(overlay, { opacity: "0" });
  });
</script>

<main class="relative select-none">
  <!-- <img
    src="public/lock_screen_simple.png"
    alt="ok"
    class="w-full h-full transition-all duration-150 ease-in-out pointer-events-none select-none backdrop-blur-md"
  /> -->
  <div
    bind:this={overlay}
    id="overlay"
    class="w-full h-full transition-all duration-150 ease-in-out pointer-events-none select-none bg-black/15 backdrop-blur-md"
  ></div>
  <div
    bind:this={buttonElement}
    class="fixed flex items-center justify-center overflow-hidden rounded-full cursor-pointer select-none w-14 h-14 bottom-8 right-8 bg-black/50 no-touch-action backdrop-blur-3xl"
    on:pointerdown={handlePointerDown}
  >
    <svg
      bind:this={icon}
      id="timer"
      width="25"
      height="25"
      viewBox="0 0 25 25"
      fill="none"
      xmlns="http://www.w3.org/2000/svg"
    >
      <path
        d="M12.5 25C5.607 25 0 19.393 0 12.5C0 9.209 1.334 5.988 3.661 3.661C3.75384 3.56816 3.86407 3.49451 3.98537 3.44426C4.10668 3.39401 4.2367 3.36815 4.368 3.36815C4.4993 3.36815 4.62932 3.39401 4.75063 3.44426C4.87193 3.49451 4.98216 3.56816 5.075 3.661C5.16784 3.75384 5.24149 3.86407 5.29174 3.98537C5.34199 4.10668 5.36785 4.2367 5.36785 4.368C5.36785 4.4993 5.34199 4.62932 5.29174 4.75063C5.24149 4.87193 5.16784 4.98216 5.075 5.075C3.11294 7.04897 2.00808 9.7168 2 12.5C2 18.29 6.71 23 12.5 23C18.29 23 23 18.29 23 12.5C23 7.047 18.823 2.552 13.5 2.047V7C13.5 7.26522 13.3946 7.51957 13.2071 7.70711C13.0196 7.89464 12.7652 8 12.5 8C12.2348 8 11.9804 7.89464 11.7929 7.70711C11.6054 7.51957 11.5 7.26522 11.5 7V1C11.5 0.734784 11.6054 0.48043 11.7929 0.292893C11.9804 0.105357 12.2348 0 12.5 0C19.393 0 25 5.607 25 12.5C25 19.393 19.393 25 12.5 25Z"
        fill="white"
      />
      <path
        d="M6.793 8.207C6.60549 8.01949 6.50015 7.76518 6.50015 7.5C6.50015 7.23482 6.60549 6.98051 6.793 6.793C6.98051 6.60549 7.23482 6.50015 7.5 6.50015C7.76518 6.50015 8.01949 6.60549 8.207 6.793L13.914 11.086C14.105 11.2705 14.2574 11.4912 14.3622 11.7352C14.467 11.9792 14.5222 12.2416 14.5245 12.5072C14.5268 12.7728 14.4762 13.0361 14.3756 13.2819C14.2751 13.5277 14.1266 13.751 13.9388 13.9388C13.751 14.1266 13.5277 14.2751 13.2819 14.3756C13.0361 14.4762 12.7728 14.5268 12.5072 14.5245C12.2416 14.5222 11.9792 14.467 11.7352 14.3622C11.4912 14.2574 11.2705 14.105 11.086 13.914L6.793 8.207Z"
        fill="white"
      />
    </svg>

    <div bind:this={sliderElement} class="select-none slider-container">
      {#each segments as segment, i}
        <div
          class="slider-segment select-none {i === totalSegments - 1
            ? 'border-none'
            : 'border-t-[0.7px] border-[#FFFFFF38]'} "
          class:filled={segment === 1}
        ></div>
      {/each}
    </div>
  </div>

  <p
    class="fixed text-base text-center text-white -translate-x-1/2 select-none top-12 left-1/2"
    bind:this={showTime}
  >
    Set timer for:<br />
    <span class="text-3xl font-medium "
      >{Math.round(sliderValue * 120)} minutes</span
    >
  </p>

  <button
    bind:this={startButton}
    class="fixed bottom-[70px] left-1/2 transform -translate-x-1/2 bg-[#30D158] text-white w-20  text-lg aspect-square rounded-full select-none"
  >
    Start
  </button>
</main>

<style>
  main {
    /* Take up the full height and width of the viewport */
    width: 100vw;
    height: 100vh;
    /* Ensure content is not hidden by the notch or home indicator */
    padding-top: env(safe-area-inset-top);
    padding-bottom: env(safe-area-inset-bottom);
    padding-left: env(safe-area-inset-left);
    padding-right: env(safe-area-inset-right);
    /* Remove default margin */
    margin: 0;
    /* Adjust according to your needs */
    background-image: url("/lock_screen_simple.png");
    background-size: cover;
    background-position: center;
  }
  .no-touch-action {
    touch-action: none;
  }
  .slider-container {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    display: flex;
    flex-direction: column-reverse;
  }
  .slider-segment {
    flex: 1;
    width: 100%;
    background-color: #00000046;
    /* border-top: 1px solid #FFFFFF38; */
    outline: 1px solid transparent;
    /* backdrop-filter: blur(10px); */
    /* margin-top: 1px; */
    transition: background-color 0.1s ease;
  }
  .slider-segment.filled {
    background-color: #ffffff9f;
    border-top: 0.7px solid #00000051;
  }
  .select-none {
    user-select: none;
  }
</style>
