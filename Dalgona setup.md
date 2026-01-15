By using Google's `<model-viewer>`, you leverage the native AR viewers built into iOS and Android, ensuring the Nescafé mug stays anchored to the table without jittery custom code.

Here is your production guide for the **"Tap-to-Advance" Nescafé Dalgona Experience**.

### 1. The 3D Asset Strategy: "One File, Multiple Clips"

Instead of swapping 3D models (which causes flickering/loading bars in AR), you will export **one single `.glb` file** that contains the entire scene and all props.

You will separate the actions into **Animation Clips** (Takes).

**Required Animation Sequence (Single Timeline Track):**

The animation uses a single track (`animation_0`) with keyframe spans:

1. **Step 1: The Mix** - Frames 1-80: Nescafé + Water into bowl
2. **Step 2: Whip It** - Frames 80-140: Whisking into foam
3. **Step 3: The Base** - Frames 140-200: Milk into mug
4. **Step 4: Transfer** - Frames 200-240: Bowl into mug
5. **Step 5: Final Touch** - Frames 240-300: Add ice

**Interactive Objects (Mesh Names):**
- Step 1: `Nescafe.BTM`, `Nescafe.TOP (Copy)`, `pot`, `handle`
- Step 2: `whisk`
- Step 3: `jug`
- Step 4: `mixing.bowl`
- Step 5: `scoop`, `ice.bowl`

**C4D Export Tip:**
Create a single animation track (`animation_0`) with all steps on one timeline. Ensure frame transitions are smooth (last frame of one step matches first frame of next). When exporting to **glTF/GLB**, ensure "Export Animation" is checked. The code will control playback using keyframe ranges.

**Important:** Object names in your GLB must match exactly (case-sensitive): `Nescafe.BTM`, `Nescafe.TOP (Copy)`, `pot`, `handle`, `whisk`, `jug`, `mixing.bowl`, `scoop`, `ice.bowl`

---

### 2. The Code Template

Copy and paste this into an `index.html` file. This script acts as your "Director," waiting for a user tap to play the next clip in your sequence.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Nescafé Dalgona AR</title>
    <script type="module" src="https://ajax.googleapis.com/ajax/libs/model-viewer/3.4.0/model-viewer.min.js"></script>
    <style>
        body { margin: 0; padding: 0; font-family: 'Arial', sans-serif; }
        model-viewer { width: 100vw; height: 100vh; background-color: #f0f0f0; }
        
        /* The UI Overlay */
        #ui-container {
            position: absolute;
            bottom: 30px;
            left: 50%;
            transform: translateX(-50%);
            text-align: center;
            width: 100%;
            pointer-events: none; /* Let taps pass through to the model */
        }
        
        .instruction-card {
            background: white;
            padding: 15px 25px;
            border-radius: 30px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.2);
            display: inline-block;
            font-weight: bold;
            color: #333;
            transition: opacity 0.3s;
        }

        /* Hide UI when AR is initializing */
        model-viewer:not([loaded]) #ui-container { opacity: 0; }
    </style>
</head>
<body>

    <model-viewer 
        id="coffee-experience"
        src="assets/Nescafe_Dalgona_Master.glb"
        ios-src="assets/Nescafe_Dalgona_Master.usdz"
        ar 
        ar-modes="webxr scene-viewer quick-look" 
        camera-controls 
        disable-zoom
        shadow-intensity="1"
        animation-name="Idle" 
        autoplay>

        <button slot="ar-button" style="background-color: #E82329; border-radius: 4px; border: none; position: absolute; top: 16px; right: 16px; color: white; padding: 10px 20px;">
            👋 Start AR Experience
        </button>

        <div id="ui-container">
            <div id="instruction-text" class="instruction-card">Tap to start mixing!</div>
        </div>

    </model-viewer>

    <script>
        const modelViewer = document.querySelector('#coffee-experience');
        const uiText = document.querySelector('#instruction-text');

        // Define your Animation Sequence
        // These names MUST match your C4D Take names exactly
        const sequence = [
            { anim: 'Step1_Mix',      text: 'Adding Nescafé & Water...' },
            { anim: 'Step2_Whip',     text: 'Whipping into foam...' },
            { anim: 'Step3_Mug',      text: 'Preparing the milk...' },
            { anim: 'Step4_Transfer', text: 'Top it off!' },
            { anim: 'Idle_Finished',  text: 'Enjoy your Dalgona!' } // A final loop where steam rises
        ];

        let currentStep = -1; // -1 means we haven't started yet
        let isAnimating = false;

        // Listen for User Taps
        // Note: We attach click to the window to capture taps anywhere
        window.addEventListener('click', (event) => {
            
            // Prevent tapping if interaction shouldn't happen yet
            if (isAnimating) return; 
            if (currentStep >= sequence.length - 1) return; // Recipe done

            // Advance Step
            currentStep++;
            playStep(currentStep);
        });

        function playStep(index) {
            const stepData = sequence[index];
            
            isAnimating = true;
            uiText.textContent = stepData.text; // Update UI
            
            // Tell Model Viewer to play the specific clip
            modelViewer.animationName = stepData.anim;
            modelViewer.play();
            
            // Loop Logic:
            // By default, model-viewer loops animations. 
            // We want to play ONCE, then stop (or switch to an idle loop).
            // We use the 'loop' property to control this, or listen for finish.
            modelViewer.loop = false; 
        }

        // Listen for when an animation finishes
        modelViewer.addEventListener('finished', () => {
            isAnimating = false;
            
            // Logic to update UI for the NEXT interaction
            if (currentStep < sequence.length - 1) {
                uiText.textContent = "Tap to continue";
            } else {
                uiText.textContent = "Done! Tap to restart.";
                currentStep = -1; // Reset logic for demo purposes
            }
        });

    </script>
</body>
</html>

```

### 3. Critical Workflow Details

#### A. Handling the "Pause" State in C4D

When an animation finishes in AR, the object will snap back to the start frame *unless* you handle it.

* **The Problem:** `Step1_Mix` ends. If the animation loops or stops, the sachet might snap back to the table.
* **The Fix:** In C4D, ensure the **last frame** of `Step1_Mix` matches the **first frame** of `Step2_Whip`.
* **Alternative:** When `Step1_Mix` finishes in the JS code, you can immediately switch `modelViewer.animationName` to a generic "Idle_Wait" loop that keeps the sachet empty and the water in the bowl.

#### B. Lighting (Environment Map)

`model-viewer` comes with default HDR lighting, but for glass and coffee, you want specific reflections.

* Add the `environment-image` tag to the HTML to load a custom HDR (e.g., a kitchen interior).
```html
<model-viewer ... environment-image="assets/kitchen_environment.hdr">

```



#### C. Testing

You don't need a server to test this logic.

1. Use the **Glitch.com** or **CodeSandbox** online editors.
2. Upload your `.glb` there.
3. Paste the HTML code.
4. Open the URL on your phone.

### Summary of Tasks for You:

1. **C4D:** Create one scene. Animate the whole sequence.
2. **C4D:** Use the "Take System" to slice that timeline into named clips (Step1, Step2, etc.).
3. **Export:** GLB with "Export Takes" enabled.
4. **Web:** Drop the file into the code template above.

This workflow is robust, avoids third-party subscription fees, and delivers a high-quality Nescafé branded experience.