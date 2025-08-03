<script>
    import { onMount } from 'svelte';
  import { draw } from 'svelte/transition';

  // curtime to local time string without am/pm

    let curtime = $state(
        new Date()

    );

    let canvas = $state(null);
    let canvasHands = $state(null);
    let ctx = $state(null);
    let gyroScopeRotation = $state({
        alpha: 0,
        beta: 0,
        gamma: 0
    });
    let ctxHands = $state(null);
    $effect(() => {
        const interval = setInterval(() => {
            ctxHands.clearRect(0, 0, canvas.width, canvas.height);
            drawClockHands(ctxHands, canvasHands.width / 2 - 0);

            if( curtime.getSeconds() % 10 == 0) {
                let pattern = new Array(curtime.getSeconds() % 10).fill(100).concat(new Array(10 - (curtime.getSeconds() % 10)).fill(0));
                navigator.vibrate(pattern);
            }
            else{
                navigator.vibrate(10);
            }
        }, 1000);
        return () => clearInterval(interval);
    });


    function drawTickMarks(ctx, radius) {
    ctx.save();
    ctx.translate(radius, radius); // Move to center of the canvas

        ctx.clearRect(0, 0, canvas.width, canvas.height);
        ctx.fillStyle = 'black';
        ctx.font = '60px Arial';
        ctx.textAlign = 'center';
        ctx.textBaseline = 'middle';

    for (let i = 0; i < 60; i++) {
        const angle = (i * Math.PI) / 30; // 360° in radians = 2π, divide into 60 segments

        const isHourMark = i % 5 === 0;
        const tickLength = isHourMark ? 40 : 20;
        const tickWidth = isHourMark ? 5 : 3;
        const outerRadius = radius -10;
        const innerRadius = radius - tickLength;

        ctx.lineCap = "round"; // Rounded ends for the ticks
        ctx.beginPath();
        ctx.lineWidth = tickWidth;
        ctx.strokeStyle = "#000"; // white ticks

        const x1 = Math.cos(angle) * outerRadius;
        const y1 = Math.sin(angle) * outerRadius;
        const x2 = Math.cos(angle) * innerRadius;
        const y2 = Math.sin(angle) * innerRadius;

        ctx.moveTo(x1, y1);
        ctx.lineTo(x2, y2);
        ctx.stroke();
    }

    ctx.restore();
    }

    function drawClockHands(_ctx, radius) {
  const now = new Date();
  const hour = now.getHours() % 12;
  const minute = now.getMinutes();
  const second = now.getSeconds();

  _ctx.save();
  _ctx.translate(radius, radius); // Center of canvas

  // Hour hand
  const hourAngle = ((hour + minute / 60) * Math.PI) / 6;
  drawHand(_ctx, hourAngle, radius * 0.85, 6, "#000");

  // Minute hand
  const minuteAngle = ((minute + second / 60) * Math.PI) / 30;
  drawHand(_ctx, minuteAngle, radius * 0.9, 4, "#CCC");

  // Second hand (red, thin, long)
  const secondAngle = (second * Math.PI) / 30;
  drawHand(_ctx, secondAngle, radius * 10, 2, "red");

  _ctx.restore();
}

function drawHand(_ctx, angle, length, width, color) {
  _ctx.beginPath();
  _ctx.lineWidth = width;
  _ctx.lineCap = "round";
  _ctx.strokeStyle = color;
  _ctx.rotate(angle);
  _ctx.moveTo(0, 0);
  _ctx.lineTo(0, -length);
  _ctx.stroke();
  _ctx.rotate(-angle);
}


    
    onMount(() => {
        ctx = canvas.getContext('2d');
        ctxHands = canvasHands.getContext('2d');
        ctx.fillStyle = 'white';
        ctx.fillRect(0, 0, canvas.width, canvas.height);

        window.addEventListener("deviceorientation", (event) => {
            gyroScopeRotation.alpha = event.alpha; // Rotation around z-axis
            gyroScopeRotation.beta = event.beta;   // Tilt front-back
            gyroScopeRotation.gamma = event.gamma; // Tilt left-right

});

        drawTickMarks(ctx, canvas.width / 2);
    });
</script>


<div class="flex w-full h-full justify-center items-center">
    <div class="rounded-full  size-112 flex justify-center items-center flex-col">

        <!-- <p>{curtime}</p> -->
         <div class="w-full h-full relative flex justify-center items-center">
            <!--
            <div class="absolute z-30 bg-white size-[200px] flex justify-center text-center items-center rounded-full p-2 text-5xl">
                {#each Object.keys(gyroScopeRotation) as gyroParam}
                    <span>{gyroParam}:{gyroScopeRotation.alpha}</span><br />
                {/each}
            </div>
        -->
            <div class="liquidGlass z-20 bg-slate-200/20 font-bold text-shadow-[1px_1px_1px_rgba(255,255,255,255.5)] text-slate-600 size-[200px] flex justify-center text-center items-center rounded-full p-2 text-5xl"><span>{curtime.toLocaleTimeString()}</span></div>
            <canvas class="rounded-full size-[350px] z-10 absolute" bind:this={canvasHands} width="500" height="500" ></canvas>
            <canvas class="rounded-full border-3  border-dotted w-full h-full absolute left-[-0.7px] top-[-0.7px]" bind:this={canvas} width="500" height="500" ></canvas>
         </div>
    </div>
</div>

<style>
    .liquidGlass {
        backdrop-filter: blur(4px);
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    }
</style>
