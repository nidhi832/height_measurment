KYRA — Person Height Measurement Module
Marker-free, single-webcam real-time height estimation using MediaPipe Pose + OpenCV

What It Does
KYRA measures the real-world standing height of a person (in centimetres) using only a standard laptop or USB webcam — no depth sensors, no physical markers, no special equipment.

It uses Google's MediaPipe Pose AI to detect body keypoints in real time, computes a precise pixel-to-centimetre scale through a one-time calibration step, and displays the measured height live on screen with position guidance overlays.

Key Features
Marker-free AI detection — uses MediaPipe body pose landmarks, no QR codes or stickers needed
Anatomically-scaled head estimation — head-top is computed dynamically from the nose-to-shoulder distance, eliminating the +/-2 to 3 cm bias between short and tall people
Two calibration methods — calibrate using yourself (most accurate) or any object of known height
Real-time position guidance — on-screen POSITION: OK / STEP BACK / STEP FORWARD indicators
Posture bending detection — warns if person is leaning or bending (pauses measurement until straight)
Rolling median smoothing — 10-frame median filter eliminates measurement jitter
Persistent calibration — scale saved to config/height_calibration.json, works across sessions
Validation suite — automated accuracy sweep with PASS/FAIL report and distance sensitivity testing
Tech Stack
Library	Purpose
mediapipe	AI body pose landmark detection
opencv-python	Camera capture, frame display, ROI selection
numpy	Pixel math, rolling median
Python 3.11	Runtime
Project Structure
height/      config/         height_calibration.json     <- Auto-generated after calibration      src/         height/            __init__.py            landmarks.py            <- MediaPipe head/feet detection + posture check            height_webcam.py        <- Scale-based height calculator + calibration I/O            height_stereo.py        <- RANSAC floor-plane stereo depth estimation            target_detector.py      <- Person / ROI bounding box detectors            pipeline.py             <- Main CLI entry point            validate.py             <- Accuracy sweep + distance sensitivity tests            validation_logs/        <- Auto-generated JSON session logs

Setup
1. Install dependencies
pip install mediapipe opencv-python numpy

2. Navigate to the project root
cd C:\Users\admin\Desktop\height

3. Set Python path (do this once per CMD session)
set PYTHONPATH=.

How to Run
Step 1 - Calibrate (one-time per location/camera setup)
Option A - Calibrate using your own height (most accurate): python src\height\pipeline.py --ref-person-cm 146.0

Stand straight on the marked floor spot with full body visible
Wait for HEAD and FEET dots to appear on screen
Press SPACE to capture and save calibration
Press Q to exit
Option B - Calibrate using a reference object: python src\height\pipeline.py --ref-object-cm 27.5

Place the object vertically on the floor at the standing spot
Press SPACE to freeze the frame
Draw a box from top to bottom of the object
Press ENTER to save calibration
Replace the number with your actual height or object height in centimetres.

Step 2 - Measure Height (live webcam)
python src\height\pipeline.py --mode webcam --target person

Stand on the calibrated spot
Wait for POSITION: OK (green) on screen
Your height is displayed at the bottom of the screen
Press Q or ESC to exit
Step 3 - Validate Accuracy
python src\height\validate.py --mode webcam --known-height 146.0 --samples 20 Collects 20 frames and reports mean error, std deviation, and PASS/FAIL.

Step 4 - Distance Sensitivity Test
python src\height\validate.py --mode distance-test --known-height 146.0 Tests accuracy at close (~1m), medium (~2m), and far (~3m) distances.

How It Works
Camera Frame        |   MediaPipe Pose (33 body landmarks)        |   Head Top  = Nose - (50% x nose-to-shoulder distance)   <- scales with person size   Feet      = max Y of (ankles + heels + toes)           <- reaches true floor level        |   pixel_span = |head_y - feet_y|   height_cm  = (pixel_span / pixels_per_cm) + 1.0        |   Rolling 10-frame median -> displayed on screen

Why the anatomical head-shift matters
Previous versions used a fixed 5% of frame height as the head offset. This caused:

Short people: 24px shift overcompensated -> reading too high
Tall people: 24px shift undercompensated -> reading too low
Result: +/-2 to 3 cm bias depending on who was being measured
The fix uses 50% of the nose-to-shoulder distance as the offset. Since taller people have a larger nose-to-shoulder gap on screen, the shift scales automatically - eliminating the cross-height bias entirely.

On-Screen Indicators
Display	Meaning
POSITION: OK (green)	Standing at correct calibrated distance
POSITION: STEP BACK (orange)	Too close to camera
POSITION: STEP FORWARD (orange)	Too far from camera
POSITION: NO PERSON DETECTED	Body not visible
STAND STRAIGHT: leaning sideways (red)	Shoulder tilt detected - pausing measurement
STAND STRAIGHT: bending detected (red)	Forward bend detected - pausing measurement
Height: 146.2 cm [calibrated_scale] (green)	Live measured height
Height: Detecting... (orange)	Waiting for detection
Accuracy
Condition	Expected Error
Person self-calibration + plain background	+/-0.5 to 1.0 cm
Object calibration + same conditions	+/-1.0 to 2.0 cm
No calibration (default scale)	+/-5 to 15 cm
Tips for best accuracy
Use a plain wall background (white or grey)
Place camera at chest height, lens level (not tilted)
Ensure full body - head top to toes - is visible in frame
Stand still and straight on the exact calibrated spot
Good frontal lighting, avoid bright windows behind you
Troubleshooting
Problem	Fix
ModuleNotFoundError: No module named 'src'	Run from project root and set PYTHONPATH=.
Height: Detecting... always shown	Step back so full body is visible in frame
Height is 10-15 cm too short	Camera is tilted up; set it level at chest height
Height jumps after recalibration	Delete config/height_calibration.json and recalibrate
POSITION: STEP BACK when standing still	Recalibrate in current location
Camera not opening	Try --left-cam 1 to switch camera index
Calibration File
Saved automatically at config/height_calibration.json after any calibration:

{       "pixels_per_cm": 8.52,       "calibrated_base_y": 447,       "updated_at": "2026-07-16 10:32:00"   }

Re-run calibration whenever you move the camera or change location.

