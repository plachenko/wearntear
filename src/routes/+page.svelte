<script>
import { onMount } from 'svelte';
import { fade } from 'svelte/transition';

// curtime to local time string without am/pm

let curtime = $state(null);
let displayedTime = $state(null);
let canvas = $state(null);
let canvasHands = $state(null);
let setToVibrate = $state(false); // true to vibrate every 10 seconds, false to vibrate every second
let ctx = $state(null);
let gyroScopeRotation = $state({
    alpha: 0,
    beta: 0,
    gamma: 0
});
let handsDrawn = $state(false);
let ticksDrawn = $state(false);
let ctxHands = $state(null);
let pageLoaded = $state(false);

let accent = $state('rgba(0, 255, 0, 1)'); // default glow color

$effect(() =>{ 
    if(handsDrawn) {
        ctxHands = canvasHands.getContext('2d');
        drawClockHands(ctxHands, canvasHands.width / 2 - 0);
    }
});

function drawTickMarks(ctx, radius) {
    ctx.save();
    ctx.translate(radius, radius); // Move to center of the canvas

    ctx.clearRect(0, 0, canvas.width, canvas.height);
    ctx.fillStyle = accent;
    ctx.font = 'bold 60px Arial';
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';

    for (let i = 0; i < 60; i++) {
        const angle = (i * Math.PI) / 30; // 360° in radians = 2π, divide into 60 segments

        const isHourMark = i % 5 === 0;
        const tickLength = isHourMark ? 25 : 20;
        const tickWidth = isHourMark ? 5 : 3;
        const outerRadius = radius;
        const innerRadius = radius - tickLength;

        ctx.lineCap = "round"; // Rounded ends for the ticks
        ctx.beginPath();
        ctx.lineWidth = tickWidth;
        ctx.strokeStyle = isHourMark ? accent : "#FFF"; // green for hour marks, white for minute marks

        const x1 = Math.cos(angle) * outerRadius;
        const y1 = Math.sin(angle) * outerRadius;
        const x2 = Math.cos(angle) * innerRadius;
        const y2 = Math.sin(angle) * innerRadius;

        if (isHourMark && i % 15 === 0) {
            let hour = i / 5;
            if (hour === 0) hour = 12;

            let offset = 1.57;
            const textRadius = hour == 12 ? radius - 65 : radius - 55; // move text slightly inward
            const textX = Math.cos(angle-offset) * textRadius;
            const textY = Math.sin(angle-offset) * textRadius;

            ctx.fillText(hour.toString(), textX, textY);

            ctx.moveTo(x1, y1);
            ctx.lineTo(x2, y2);
            ctx.stroke();
        }else{
            ctx.moveTo(x1, y1);
            ctx.lineTo(x2, y2);
            ctx.stroke();
        }


    }

    ctx.restore();
}

function drawClockHands(_ctx, radius) {
    _ctx.clearRect(0, 0, canvas.width, canvas.height);
    const now = new Date();
    const hour = now.getHours() % 12;
    const minute = now.getMinutes();
    const second = now.getSeconds();

    _ctx.save();
    _ctx.translate(radius, radius); // Center of canvas

    // Hour hand
    const hourAngle = ((hour + minute / 60) * Math.PI) / 6;
    drawHand(_ctx, hourAngle, radius * 0.85, 6, "#FFF");

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


function connectToGoogleCalendar() {
    const calendarId = "a2a1054adaa540c0798d61fc97a28e54cc56c4d150819eb5db919315c2eaee3b@group.calendar.google.com";
    const apiKey = "AIzaSyCBMIGxYQJRP9t6BCcB3ZJoD6g8sXFjOyY";
    const timeMin = new Date().toISOString(); // Now
    const url = `https://www.googleapis.com/calendar/v3/calendars/${encodeURIComponent(calendarId)}/events?key=${apiKey}&timeMin=${timeMin}&maxResults=10&singleEvents=true&orderBy=startTime`;

    fetch(url)
    .then(response => response.json())
    .then(data => {
        if (data.items) {
        data.items.forEach(event => {
            console.log("Event:", event.summary);
            console.log("Start:", event.start.dateTime || event.start.date);
            console.log("End:", event.end.dateTime || event.end.date);
        });
        } else {
        console.warn("No events found or error:", data);
        }
    })
    .catch(error => console.error("Fetch error:", error));
}
    
onMount(() => {
    curtime = new Date();
    // displayedTime = curtime.toLocaleTimeString('en-US', { hour: '2-digit', minute: '2-digit',  hour12: true }).replaceAll(/\b0/g, "");
   
    connectToGoogleCalendar();

    setInterval(() => {
            curtime = new Date();
            // displayedTime = curtime.toLocaleTimeString('en-US', { hour: '2-digit', minute: '2-digit',  hour12: true }).replaceAll(/\b0/g, "");
            drawClockHands(ctxHands, canvasHands.width / 2 - 0);
        }, 1000);

    pageLoaded = true;

    window.addEventListener("scroll", (event) => {
        event.preventDefault();
    }, { passive: false });


    setTimeout(() => {
        ticksDrawn = true;
        ctx = canvas.getContext('2d');
        handsDrawn = true;
        // ctxHands = canvasHands.getContext('2d');
        // drawClockHands(ctxHands, canvasHands.width / 2 - 0);
        drawTickMarks(ctx, canvas.width / 2);
    }, 100);
});
</script>


    
<div class="flex w-full h-full justify-center items-center bg-black">
    <div class="rounded-full border-4 border-white size-112 flex justify-center items-center flex-col">

        {#if curtime}
        <div transition:fade class="w-full h-full relative flex justify-center items-center">

            
            <div 
                class="liquidGlass z-20 bg-slate-200/20 font-bold text-slate-200 size-[200px] flex justify-center text-center items-center rounded-full p-2 text-5xl">
                <span>
                    {curtime.getHours() % 12}<span class={`${curtime.getSeconds() % 2 === 0 ? 'text-slate-200' : 'text-slate-300'}`}>:</span>{curtime.getMinutes()}
                </span>
                <div class="text-sm flex flex-col justify-center items-center ml-1 mt-2">
                    {#each ['AM', 'PM'] as ampm}
                        <span class={`${(curtime.getHours() >= 12 && ampm === 'PM') || (curtime.getHours() < 12 && ampm === 'AM') ? 'text-slate-200' : 'text-slate-500'}`}>{ampm}</span>
                    {/each}
                </div>
            </div>
            
            {#if handsDrawn}
                <canvas transition:fade class="rounded-full size-[350px] z-10 absolute" bind:this={canvasHands} width="500" height="500" ></canvas>
            {/if}
            
            <canvas transition:fade class="rounded-full size-[410px] z-10 absolute" bind:this={canvas} width="500" height="500" ></canvas>

        </div>
        {:else if pageLoaded}
            <span class="text-white">Out of time.</span>
        {/if}
    </div>
</div>

<style>

    .liquidGlass {
        backdrop-filter: blur(4px);
        box-shadow: 0 0px 8px rgba(18, 238, 18, 0.589);
    }
</style>
