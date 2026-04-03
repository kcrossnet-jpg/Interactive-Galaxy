<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>galaxy</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        html, body {
            overflow: hidden;
            background: #0a0a1f;
            height: 100vh;
            width: 100vw;
        }

        .experience {
            position: fixed;
            width: 100vw;
            height: 100vh;
            top: 0;
            left: 0;
        }

        h1 {
            position: absolute;
            top: 40px;
            left: 50%;
            transform: translateX(-50%);
            font-family: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
            font-size: 3.5rem;
            font-weight: 700;
            letter-spacing: -2px;
            background: linear-gradient(90deg, #a3ff00, #ffffff);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            text-shadow: 0 0 40px rgba(163, 255, 0, 0.5);
            z-index: 100;
            pointer-events: none;
            user-select: none;
        }
    </style>
</head>
<body>
    <h1></h1>
    <div class="experience"></div>

    <script>
        document.addEventListener('DOMContentLoaded', () => {
            const container = document.querySelector('.experience');
            const canvas = document.createElement('canvas');
            container.appendChild(canvas);
            const ctx = canvas.getContext('2d');

            let width = window.innerWidth;
            let height = window.innerHeight;
            let centerX = width / 2;
            let centerY = height / 2;

            // Resize handler
            function resizeCanvas() {
                width = window.innerWidth;
                height = window.innerHeight;
                canvas.width = width;
                canvas.height = height;
                centerX = width / 2;
                centerY = height / 2;
                initStars();
            }

            window.addEventListener('resize', resizeCanvas);

            // Mouse interaction
            let mouseX = centerX;
            let mouseY = centerY;
            let mouseActive = false;

            window.addEventListener('mousemove', (e) => {
                mouseX = e.clientX;
                mouseY = e.clientY;
                mouseActive = true;
            });

            window.addEventListener('mouseleave', () => {
                mouseActive = false;
            });

            // Background stars (static + subtle twinkle)
            let stars = [];
            function initStars() {
                stars = [];
                for (let i = 0; i < 350; i++) {
                    stars.push({
                        x: Math.random() * width,
                        y: Math.random() * height,
                        size: Math.random() * 2 + 0.5,
                        speed: Math.random() * 0.02 + 0.005
                    });
                }
            }

            // Particle class (physics-based orbiting + mouse gravity)
            class Particle {
                constructor() {
                    this.reset();
                }

                reset() {
                    const angle = Math.random() * Math.PI * 2;
                    // Start between inner edge and outer edge of the disk
                    this.radius = 65 + Math.random() * 380;
                    this.x = centerX + Math.cos(angle) * this.radius;
                    this.y = centerY + Math.sin(angle) * this.radius;

                    // Circular orbit velocity (inner particles move faster)
                    const GM = 14500; // Central black hole gravity strength
                    const v = Math.sqrt(GM / this.radius);
                    this.vx = -Math.sin(angle) * v;
                    this.vy = Math.cos(angle) * v;

                    // Color: inner = bright pink/white, outer = deep purple/blue
                    const norm = (this.radius - 65) / 380; // 0 = inner, 1 = outer
                    const hue = 325 - (norm * 75);        // 325° (pink) → 250° (purple-blue)
                    const lightness = 78 + (1 - norm) * 18; // Inner brighter
                    this.color = `hsl(${hue}, 100%, ${lightness}%)`;

                    this.size = Math.random() * 3 + 1.2;
                }

                update() {
                    // Vector to center (black hole)
                    let dx = centerX - this.x;
                    let dy = centerY - this.y;
                    let distSq = dx * dx + dy * dy;
                    let dist = Math.sqrt(distSq);

                    // Sucked into black hole → respawn
                    if (dist < 32) {
                        this.reset();
                        return;
                    }

                    // Central gravity (inverse square law)
                    const GM = 14500;
                    let force = GM / (distSq + 100); // +100 avoids extreme near-center values
                    let fx = (dx / dist) * force;
                    let fy = (dy / dist) * force;

                    // Mouse gravity (pulls particles toward cursor)
                    if (mouseActive) {
                        let mdx = mouseX - this.x;
                        let mdy = mouseY - this.y;
                        let mdistSq = mdx * mdx + mdy * mdy;
                        if (mdistSq > 10) {
                            let mdist = Math.sqrt(mdistSq);
                            // Stronger when close, weaker when far
                            let mforce = 9500 / (mdistSq + 400);
                            fx += (mdx / mdist) * mforce;
                            fy += (mdy / mdist) * mforce;
                        }
                    }

                    // Apply forces
                    this.vx += fx * 0.8;
                    this.vy += fy * 0.8;

                    // Very light damping so orbits stay energetic but don't fly away
                    this.vx *= 0.985;
                    this.vy *= 0.985;

                    // Update position
                    this.x += this.vx;
                    this.y += this.vy;
                }

                draw() {
                    // Glow effect
                    ctx.shadowBlur = 12;
                    ctx.shadowColor = this.color;
                    ctx.fillStyle = this.color;
                    ctx.fillRect(this.x, this.y, this.size, this.size);
                }
            }

            // Create particles
            let particles = [];
            function initParticles() {
                particles = [];
                const numParticles = 1450; // Dense enough for beautiful rings
                for (let i = 0; i < numParticles; i++) {
                    particles.push(new Particle());
                }
            }

            // Animation loop
            function animate() {
                // Semi-transparent fade creates glowing motion trails
                ctx.fillStyle = 'rgba(10, 10, 30, 0.13)';
                ctx.fillRect(0, 0, width, height);

                // Draw twinkling background stars
                ctx.shadowBlur = 0;
                for (let i = 0; i < stars.length; i++) {
                    const star = stars[i];
                    const alpha = 0.6 + Math.sin(Date.now() * star.speed) * 0.4;
                    ctx.globalAlpha = alpha;
                    ctx.fillStyle = '#ffffff';
                    ctx.fillRect(star.x, star.y, star.size, star.size);
                }
                ctx.globalAlpha = 1;

                // Update + draw all particles
                for (let i = 0; i < particles.length; i++) {
                    particles[i].update();
                    particles[i].draw();
                }

                // Draw the black hole (event horizon)
                // Outer soft glow
                ctx.shadowBlur = 45;
                ctx.shadowColor = '#ffccff';
                ctx.fillStyle = 'rgba(255, 255, 255, 0.15)';
                ctx.beginPath();
                ctx.arc(centerX, centerY, 48, 0, Math.PI * 2);
                ctx.fill();

                // Main black sphere
                ctx.shadowBlur = 0;
                ctx.fillStyle = '#000000';
                ctx.beginPath();
                ctx.arc(centerX, centerY, 35, 0, Math.PI * 2);
                ctx.fill();

                // Tiny inner dark core
                ctx.fillStyle = '#111111';
                ctx.beginPath();
                ctx.arc(centerX, centerY, 16, 0, Math.PI * 2);
                ctx.fill();

                requestAnimationFrame(animate);
            }

            // Initialize everything
            resizeCanvas();
            initStars();
            initParticles();
            animate();

            // Console message for fun
            console.log('%cInteractive Galaxy ready! ✨ Move your mouse to bend spacetime.', 'color:#a3ff00; font-family:monospace');
        });
    </script>
</body>
</html>
