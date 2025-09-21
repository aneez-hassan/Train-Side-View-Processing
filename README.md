# Train-Side-View-Processing
Project Structure
train-sideview-assignment/
│── train_processing.py      # main code
│── requirements.txt         # dependencies
│── README.md                # documentation
│── sample_output/           # keep some results (optional)
train_processing.py
"""
Train Wagon Coach Detection & Coverage Report
Author: Aneez Hassan
"""

import cv2, os, numpy as np
from reportlab.platypus import SimpleDocTemplate, Paragraph, Spacer, Image
from reportlab.lib.pagesizes import A4
from reportlab.lib.styles import getSampleStyleSheet

def process_video(input_video, output_dir="Processed_Video", report_path="Final_Report.pdf"):
    cap = cv2.VideoCapture(input_video)
    fps = int(cap.get(cv2.CAP_PROP_FPS))
    length = int(cap.get(cv2.CAP_PROP_FRAME_COUNT))

    os.makedirs(output_dir, exist_ok=True)

    prev_frame = None
    frame_num = 0
    coach_id = 1
    change_threshold = 1e7  # tune based on video
    coach_folders = []

    while True:
        ret, frame = cap.read()
        if not ret:
            break

        gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
        gray = cv2.GaussianBlur(gray, (5,5), 0)

        if prev_frame is not None:
            diff = cv2.absdiff(gray, prev_frame)
            score = np.sum(diff)

            if score > change_threshold:
                coach_id += 1

        # Save every 2 seconds
        if frame_num % (fps*2) == 0:
            coach_folder = os.path.join(output_dir, f"coach_{coach_id}")
            os.makedirs(coach_folder, exist_ok=True)
            frame_path = os.path.join(coach_folder, f"{frame_num}.jpg")
            cv2.imwrite(frame_path, frame)

            if coach_folder not in coach_folders:
                coach_folders.append(coach_folder)

        prev_frame = gray
        frame_num += 1

    cap.release()
    print(f"✅ Coaches detected: {coach_id}")

    # Generate PDF report
    doc = SimpleDocTemplate(report_path, pagesize=A4)
    styles = getSampleStyleSheet()
    story = []

    story.append(Paragraph("🚆 Train Coverage Report", styles["Title"]))
    story.append(Paragraph(f"Total Coaches Detected: {coach_id}", styles["Normal"]))
    story.append(Spacer(1, 20))

    for folder in coach_folders:
        story.append(Paragraph(f"{os.path.basename(folder)}", styles["Heading2"]))
        for file in sorted(os.listdir(folder))[:3]:
            if file.endswith(".jpg"):
                story.append(Image(os.path.join(folder, file), width=400, height=200))
        story.append(Spacer(1, 12))

    doc.build(story)
    print(f"📄 Report saved: {report_path}")


if __name__ == "__main__":
    # Example run (update path for your video)
    process_video("train_side_view.mp4")
requirements.txt
opencv-python
numpy
reportlab
README.md
# 🚆 Train Wagon Coach Detection & Report

## 📌 Assignment Objective
This project processes a side-view video of a train to:
- Detect the number of coaches
- Split the video into per-coach frames
- Save structured output folders
- Generate a PDF coverage report with sample frames

---

## ⚙️ Installation
```bash
git clone https://github.com/<your-username>/train-sideview-assignment.git
cd train-sideview-assignment
pip install -r requirements.txt
