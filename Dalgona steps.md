Here is your complete, step-by-step master guide to building the **Nescafé Dalgona WebAR Experience** from scratch. This workflow moves from the 3D asset creation in Cinema 4D to the final code deployment.

### Phase 1: The 3D Asset Creation (Cinema 4D)

This is the most critical part. If the GLB file isn't set up correctly with "Takes," the code won't work.

**1. The Scene Setup**

* Model all your objects (Bowl, Mug, Whisk, Sachet, etc.) in a single scene.
* **Materials:** Use standard PBR materials (Albedo, Roughness, Normal). Do not use C4D-specific shaders (like Noise or Layer) as they won't export. Bake textures if necessary.

**2. The Animation Strategy (The "Take" System)**
Instead of one long timeline, you will split your animations into "Clips" using the **Take System**.

* **Open the Take Manager** (Window > Take Manager).
* **Create the following Takes:**
* `Idle`: (Frames 0-1) A static pose of the empty bowl/mug.
* `Step1_Mix`: (e.g., Frames 0-60) Sachet pours powder, water spoon dumps water.
* `Step2_Whip`: (e.g., Frames 60-120) Whisk spins, liquid morphs into foam.
* `Step3_Mug`: (e.g., Frames 120-180) Ice falls into the mug, milk rises.
* `Step4_Transfer`: (e.g., Frames 180-240) Foam blob flies from bowl to mug.
* `Final_Loop`: (e.g., Frames 240-300) Steam rising, slight rotation of the mug.



**3. Exporting to GLB**

* Go to **File > Export > glTF 2.0 (.glb)**.
* **General Settings:**
* Format: Binary (.glb).
* Textures: Checked.


* **Animation Settings:**
* **Export Animation:** Checked.
* **Takes:** Select "Export All Takes" (or manually check the ones you created).


* Click Export. Name it `nescafe_dalgona.glb`.

---

### Phase 2: Project Setup (Boilerplate)

Create a folder on your computer named `nescafe-ar`. Inside, create this structure:

```text
nescafe-ar/
├── index.html        (The logic and UI)
└── assets/
    ├── nescafe_dalgona.glb   (Your 3D file)
    └── env.hdr               (Optional: Lighting map)

```

---

### Phase 3: The Code (index.html)

Copy this entire block into your `index.html`. This includes the HTML structure, the CSS styling, and the JavaScript logic to handle the "Tap-to-Advance" gameplay.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Nescafé Dalgona AR</title>
    <script type="module" src="https://ajax.googleapis.com/ajax/libs/model-viewer/3.4.0/model-viewer.min.js"></script>
    
    <style>
        /* --- CSS STYLING --- */
        body { margin: 0; padding: 0; font-family: 'Helvetica', sans-serif; overflow: hidden; }
        
        /* The AR Window */
        model-viewer {
            width: 100vw;
            height: 100vh;
            background-color: #f5f5f5; /* Fallback color */
            --poster-color: #f5f5f5;
        }

        /* The UI Overlay Container */
        #ui-layer {
            position: absolute;
            bottom: 40px;
            left: 0;
            width: 100%;
            text-align: center;
            pointer-events: none; /* Let taps pass through to the 3D model */
            z-index: 100;
        }

        /* The Branding / Instruction Card */
        .card {
            display: inline-block;
            background: white;
            padding: 16px 24px;
            border-radius: 50px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.15);
            transform: translateY(0);
            transition: all 0.3s cubic-bezier(0.25, 0.8, 0.25, 1);
        }

        .card h2 {
            margin: 0;
            font-size: 18px;
            color: #E82329; /* Nescafe Red */
            text-transform: uppercase;
            letter-spacing: 1px;
        }

        .card p {
            margin: 4px 0 0 0;
            font-size: 14px;
            color: #555;
        }

        /* Hide UI while loading */
        model-viewer:not([loaded]) #ui-layer { opacity: 0; }

        /* Custom AR Button Styling */
        #ar-button {
            position: absolute;
            top: 20px;
            right: 20px;
            background: #E82329;
            color: white;
            border: none;
            padding: 12px 20px;
            border-radius: 8px;
            font-weight: bold;
            box-shadow: 0 4px 10px rgba(0,0,0,0.2);
            z-index: 200;
        }
    </style>
</head>
<body>

    <model-viewer 
        id="dalgona-viewer"
        src="assets/nescafe_dalgona.glb"
        ar
        ar-modes="webxr scene-viewer quick-look"
        camera-controls
        disable-zoom
        shadow-intensity="1.2"
        environment-image="neutral" 
        tone-mapping="commerce"
        animation-name="Idle">
        
        <button slot="ar-button" id="ar-button">
            View in your space
        </button>
    </model-viewer>

    <div id="ui-layer">
        <div class="card" id="info-card">
            <h2 id="step-title">Welcome</h2>
            <p id="step-desc">Tap anywhere to start cooking!</p>
        </div>
    </div>

    <script>
        const viewer = document.querySelector('#dalgona-viewer');
        const titleText = document.querySelector('#step-title');
        const descText = document.querySelector('#step-desc');
        const card = document.querySelector('#info-card');

        // GAMEPLAY CONFIGURATION
        // Map your C4D Take Names to User Instructions here
        const steps = [
            { 
                clip: 'Step1_Mix', 
                title: 'Step 1: The Mix', 
                desc: 'Adding Nescafé 3in1 & Water...' 
            },
            { 
                clip: 'Step2_Whip', 
                title: 'Step 2: Whip It', 
                desc: 'Whipping into golden foam...' 
            },
            { 
                clip: 'Step3_Mug', 
                title: 'Step 3: The Base', 
                desc: 'Pouring fresh milk over ice...' 
            },
            { 
                clip: 'Step4_Transfer', 
                title: 'Step 4: The Finish', 
                desc: 'Topping it off with the Dalgona foam!' 
            },
            { 
                clip: 'Final_Loop', 
                title: 'Ready to Serve', 
                desc: 'Enjoy your Nescafé Dalgona!' 
            }
        ];

        let currentStepIndex = -1; // Starts before the first step
        let isAnimating = false;

        // 1. LISTEN FOR TAPS
        // We listen on 'click' which works for both Mouse (Desktop) and Tap (Mobile)
        document.addEventListener('click', (event) => {
            
            // Ignore taps if we are currently playing an animation
            if (isAnimating) return;
            
            // Ignore taps if clicking the AR button
            if (event.target.id === 'ar-button') return;

            // If we have steps left, advance
            if (currentStepIndex < steps.length - 1) {
                playNextStep();
            } else {
                // Optional: Restart logic
                resetExperience();
            }
        });

        // 2. PLAY FUNCTION
        function playNextStep() {
            currentStepIndex++;
            const stepData = steps[currentStepIndex];

            // Update UI
            titleText.innerText = stepData.title;
            descText.innerText = stepData.desc;
            isAnimating = true;

            // Trigger Animation
            viewer.animationName = stepData.clip;
            viewer.play();
            
            // Important: We only loop the FINAL step. All others play once.
            if (currentStepIndex === steps.length - 1) {
                viewer.loop = true;
            } else {
                viewer.loop = false;
            }
        }

        // 3. LISTEN FOR COMPLETION
        viewer.addEventListener('finished', () => {
            // This fires when a non-looping animation finishes
            isAnimating = false;
            
            // Update UI to prompt user for the next action
            if (currentStepIndex < steps.length - 1) {
                descText.innerText = "Tap to continue ->";
            }
        });

        // 4. RESET LOGIC (Optional)
        function resetExperience() {
            currentStepIndex = -1;
            titleText.innerText = "Replay?";
            descText.innerText = "Tap to make another one.";
            viewer.animationName = "Idle";
        }

    </script>
</body>
</html>

```

---

### Phase 4: How to Test & Run It

You cannot simply double-click `index.html` to run it. AR requires a server environment to load the `.glb` file and access the camera.

**Option A: VS Code (Easiest Local Method)**

1. Open your `nescafe-ar` folder in **Visual Studio Code**.
2. Install the extension **"Live Server"**.
3. Right-click `index.html` and select **"Open with Live Server"**.
4. It will open in your desktop browser. You can click to test the animation flow.

**Option B: Testing on Mobile (AR)**
To see the AR button appear, you need **HTTPS** (secure connection) or "Port Forwarding."

1. **Quickest Way:** Use **ngrok** (free tool) to tunnel your localhost to the web.
* Run Live Server (e.g., port 5500).
* Terminal: `ngrok http 5500`.
* Send the `https://...` link to your phone.


2. **Deployment:** Upload the folder to **GitHub Pages** or **Netlify** (drag and drop). These provide HTTPS automatically, enabling the AR button on iOS/Android immediately.

### Troubleshooting Tips

* **"Animation not playing?"** Check that the `clip: 'Name'` in the JS array matches your C4D Take name *exactly* (case sensitive).
* **"Model looks dark?"** Ensure you have lights in your C4D scene (baked) or add `environment-image="legacy"` to the model-viewer tag for default lighting.
* **"Object snapping back?"** Ensure the first frame of Step 2 matches the last frame of Step 1 in Cinema 4D.