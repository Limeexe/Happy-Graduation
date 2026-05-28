<script lang="ts">
    import { onMount } from 'svelte';
    import gsap from 'gsap';
    import confetti from 'canvas-confetti';

    // DOM Element bindings with TypeScript types
    let wrapper: HTMLDivElement;
    let seal: HTMLDivElement;
    let replayBtn: HTMLButtonElement;
    let photocard: HTMLDivElement;
    let envFlap: HTMLDivElement;

    // Component State
    let isAnimating: boolean = false;
    let isOpen: boolean = false;
    let tl: gsap.core.Timeline;
    let sealAnimation: string = 'pulse-ring 2s infinite';

    onMount(() => {
        // Initialize GSAP timeline
        tl = gsap.timeline({ paused: true });
    });

    function buildTimeline(): void {
        tl.clear(); // Clear existing in case of replay
        
        // 1. Open the flap
        tl.to(envFlap, { rotateX: 180, duration: 0.7, ease: "power2.inOut" }, 0);
        
        // 2. Fade out the seal right before the flap goes fully upside down
        tl.to(seal, { opacity: 0, duration: 0.2 }, 0.2);

        // 3. Critical Step: Bring photocard in front of the flap before sliding
        tl.set(photocard, { zIndex: 45 }, 0.7);

        // 4. Slide photocard UP out of the envelope
        tl.to(photocard, { y: -180, duration: 0.8, ease: "power2.out" }, 0.8);

        // 5. Envelope falls down and fades out
        tl.to('.envelope-element', { y: 300, opacity: 0, duration: 0.8, ease: "power2.in" }, 1.2);

        // 6. Photocard moves down to center screen, scales up, and tilts slightly
        tl.to(photocard, { 
            y: 20, 
            scale: window.innerWidth < 400 ? 1.05 : 1.15,
            rotation: -2,
            boxShadow: "0 25px 50px rgba(0,0,0,0.3)",
            duration: 0.8, 
            ease: "back.out(1.2)" 
        }, 1.4);

        // 7. Fire Confetti
        tl.call(fireConfetti, [], 1.6);

        // 8. Show Replay Button inside the card
        tl.to(replayBtn, {
            opacity: 1,
            y: 0,
            pointerEvents: 'auto',
            duration: 0.5,
            ease: "power1.out"
        }, 2.2);
    }

    function handleEnvelopeClick(): void {
        if (isAnimating || isOpen) return;
        isAnimating = true;
        isOpen = true;
        
        // Remove pulse animation from seal using Svelte style binding
        sealAnimation = 'none';
        
        buildTimeline();
        tl.play().then(() => {
            isAnimating = false;
        });
    }

    function handleReplay(e: MouseEvent): void {
        if (isAnimating) return;
        isAnimating = true;
        
        // Fade out card and button immediately
        gsap.to(photocard, { opacity: 0, duration: 0.4, onComplete: () => {
            // Reset all properties to starting states
            gsap.set(envFlap, { rotateX: 0 });
            gsap.set(seal, { opacity: 1 });
            gsap.set(photocard, { y: 0, scale: 1, rotation: 0, zIndex: 20, opacity: 1, boxShadow: "0 15px 35px rgba(0,0,0,0.25)" });
            gsap.set('.envelope-element', { y: 0, opacity: 1 });
            gsap.set(replayBtn, { opacity: 0, y: 10, pointerEvents: 'none' });
            
            // Restore pulse animation
            sealAnimation = 'pulse-ring 2s infinite';
            
            isOpen = false;
            isAnimating = false;
        }});
    }

    function fireConfetti(): void {
        const duration: number = 3000;
        const end: number = Date.now() + duration;

        (function frame() {
            confetti({
                particleCount: 6,
                angle: 60,
                spread: 55,
                origin: { x: 0, y: 0.7 },
                colors: ['#2563eb', '#fbbf24', '#ffffff', '#1e40af', '#60a5fa']
            });
            
            confetti({
                particleCount: 6,
                angle: 120,
                spread: 55,
                origin: { x: 1, y: 0.7 },
                colors: ['#2563eb', '#fbbf24', '#ffffff', '#1e40af', '#60a5fa']
            });

            if (Date.now() < end) {
                requestAnimationFrame(frame);
            }
        }());
    }
</script>

<main class="scene-container">
    <div id="scene">
        <div class="envelope-wrapper" bind:this={wrapper} on:click={handleEnvelopeClick}>
            
            <div class="env-part env-back envelope-element"></div>
            
            <div class="photocard" bind:this={photocard}>
                <div class="photocard-img-wrapper mb-4">
                    <img src="https://images.unsplash.com/photo-1523050854058-8df90110c9f1?q=80&w=800&auto=format&fit=crop" alt="Graduation Celebration" class="photocard-img">
                    <div class="absolute inset-0 bg-gradient-to-t from-black/60 to-transparent flex items-end p-3">
                        <span class="text-white font-bold text-lg tracking-wide uppercase">Class of 2026</span>
                    </div>
                </div>
                
                <div class="flex-1 flex flex-col items-center text-center justify-between">
                    <div>
                        <h2 class="text-2xl font-bold text-gray-800 mb-2">You Did It! 🎉</h2>
                        <p class="text-gray-600 text-sm md:text-base leading-relaxed px-2">
                            All your hard work and late nights have finally paid off. Wishing you endless success as you start this exciting new chapter!
                        </p>
                    </div>
                    
                    <button 
                        bind:this={replayBtn} 
                        on:click|stopPropagation={handleReplay}
                        class="replay-btn mt-4 flex items-center gap-2 text-blue-600 hover:text-blue-800 bg-blue-50 px-4 py-2 rounded-full transition-colors text-sm font-semibold"
                    >
                        <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M3 12a9 9 0 1 0 9-9 9.75 9.75 0 0 0-6.74 2.74L3 8"/><path d="M3 3v5h5"/></svg>
                        Replay
                    </button>
                </div>
            </div>

            <div class="env-part env-front envelope-element"></div>

            <div class="env-flap envelope-element" bind:this={envFlap}>
                <div class="seal" bind:this={seal} style="animation: {sealAnimation};">
                    <svg width="32" height="32" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
                        <path d="M12 4L2 9L12 14L22 9L12 4Z" fill="#b45309"/>
                        <path d="M22 9V16" stroke="#b45309" stroke-width="2" stroke-linecap="round"/>
                        <path d="M5 10.5V15.5C5 18 8.13401 20 12 20C15.866 20 19 18 19 15.5V10.5" stroke="#b45309" stroke-width="2" stroke-linecap="round"/>
                    </svg>
                </div>
            </div>
            
        </div>
    </div>
</main>

<style>
    .scene-container {
        font-family: 'Inter', sans-serif;
        background: linear-gradient(135deg, #e0e7ff 0%, #ede9fe 100%);
        margin: 0;
        padding: 0;
        height: 100vh;
        width: 100vw;
        overflow: hidden;
        display: flex;
        justify-content: center;
        align-items: center;
    }

    #scene {
        perspective: 1200px;
        width: 100%;
        height: 100%;
        display: flex;
        justify-content: center;
        align-items: center;
    }

    .envelope-wrapper {
        position: relative;
        width: 90vw;
        max-width: 340px;
        height: 480px;
        transform-style: preserve-3d;
        display: flex;
        justify-content: center;
        align-items: flex-end;
        padding-bottom: 20px;
        cursor: pointer;
    }

    .env-part {
        position: absolute;
        bottom: 20px;
        width: 100%;
        height: 400px;
        border-radius: 12px;
    }

    .env-back {
        background: linear-gradient(to bottom, #1e3a8a, #172554);
        z-index: 10;
        box-shadow: inset 0 0 40px rgba(0,0,0,0.3);
    }

    .env-front {
        background: linear-gradient(to bottom, #2563eb, #1d4ed8);
        z-index: 30;
        clip-path: polygon(0 0, 50% 35%, 100% 0, 100% 100%, 0 100%);
        box-shadow: 0 -5px 15px rgba(0,0,0,0.1);
    }

    .env-flap {
        position: absolute;
        top: 60px;
        left: 0;
        width: 100%;
        height: 220px;
        background: #2563eb;
        z-index: 40;
        transform-origin: top center;
        clip-path: polygon(0 0, 100% 0, 50% 100%);
        filter: drop-shadow(0 10px 10px rgba(0,0,0,0.2));
        display: flex;
        justify-content: center;
    }

    .photocard {
        position: absolute;
        bottom: 30px;
        width: 92%;
        height: 380px;
        background: #ffffff;
        border-radius: 16px;
        box-shadow: 0 15px 35px rgba(0,0,0,0.25);
        z-index: 20;
        display: flex;
        flex-direction: column;
        padding: 16px;
        box-sizing: border-box;
        will-change: transform;
    }

    .photocard-img-wrapper {
        width: 100%;
        height: 200px;
        border-radius: 12px;
        overflow: hidden;
        position: relative;
    }

    .photocard-img {
        width: 100%;
        height: 100%;
        object-fit: cover;
    }

    @keyframes pulse-ring {
        0% { transform: scale(0.8); box-shadow: 0 0 0 0 rgba(251, 191, 36, 0.7); }
        70% { transform: scale(1); box-shadow: 0 0 0 15px rgba(251, 191, 36, 0); }
        100% { transform: scale(0.8); box-shadow: 0 0 0 0 rgba(251, 191, 36, 0); }
    }

    .seal {
        position: absolute;
        bottom: 20px;
        width: 60px;
        height: 60px;
        background: #fbbf24;
        border-radius: 50%;
        z-index: 50;
        display: flex;
        justify-content: center;
        align-items: center;
        box-shadow: 0 4px 10px rgba(0,0,0,0.3);
    }

    .replay-btn {
        opacity: 0;
        pointer-events: none;
        transform: translateY(10px);
    }
</style>