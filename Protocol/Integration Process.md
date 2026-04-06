# **[System Master Document] App Integration Process**

본 문서는 앱의 최초 구동 시점부터, 사용자가 이미지를 업로드하고 분석을 거쳐 최종 렌더링을 실행하기까지의 '시스템 통합 제어 흐름'을 타임라인 순으로 명세합니다.

---

## **PHASE 0. INIT (시스템 초기화 및 구조 대기)**
앱이 최초 로드될 때의 초기 상태 설정.

*   **UI 상태**: 다크모드/라이트모드 설정 로드.
*   **Canvas 초기화**: 빈 무한 캔버스(가운데 정렬)와 3x3 비율의 합성 그리드 출력.
*   **Tool Load**: 컨트롤 바는 하단 중앙에 배치되며 기본 모드는 Select (Cursor) 유지. 옵션 패널(Right Sidebar)은 닫혀있음.

---

> **[System Coordinate Constants — Architectural Front-Identification Logic]**
>
> 본 시스템의 모든 PHASE에서 공통으로 적용되는 **건축 방위 좌표계 상수**. 정면 인식의 절대적 기준.
>
> | 모서리 코드 | 위치 | 설명 |
> |---|---|---|
> | **Blue (좌전)** | Front-Left | 건축물 정면을 바라보았을 때의 왼쪽 앞 모서리 |
> | **Red (우전)** | Front-Right | 건축물 정면을 바라보았을 때의 오른쪽 앞 모서리 |
> | **Yellow (좌후)** | Back-Left | 건축물의 왼쪽 뒤 모서리 |
> | **Green (우후)** | Back-Right | 건축물의 오른쪽 뒤 모서리 |
>
> **정면(Front, 06:00)** = Blue–Red 선(Front Edge). 이 방향이 모든 PHASE의 시점 연산 원점(Origin).

---

## **PHASE 1. INGESTION (이미지 주입 및 기본 시점 분석)**
사용자가 원본 이미지의 [분석] 버튼을 클릭하는 시점. 여기서부터 AI 엔진이 부분 가동됩니다.

1.  **Image Upload Event**: 사용자가 .jpg, .png 등 건축물 이미지를 업로드.
2.  **State Reset**: 기존의 생성된 이미지(generatedImage) 및 배치도(sitePlanImage)를 초기화(Null)하고 UI를 갱신.
3.  **Image Pre-Processing — 이미지 재생성 (사용자 비노출)**: [분석] 버튼 클릭과 동시에 최초 수행.
    *   **Engine:** `gemini-3.1-flash-image-preview`
    *   **목적:** 업로드 원본의 JPEG 압축 노이즈, 렌즈 왜곡, 색보정 왜곡을 AI 정규화 거쳐 제거.
    *   **출력:** 내부 변수 `analysisImageBase64` — 캔버스에는 원본 이미지 유지, 이후 PHASE 1 분석에만 내부 사용.
    *   **실패 시:** Fallback으로 원본 이미지를 그대로 사용하며 분석 계속 진행.
4.  **Basic Viewpoint Analysis & 정보 구조화 분석 (분석 및 추출 동시 개시)**:
    *   **Engine:** `gemini-3.1-pro-preview` (MODEL_ANALYSIS)
    *   **시점 추론:** 재생성된 이미지(`analysisImageBase64`)를 기반으로 "정면(06:00) 기준점" 대비 현재 관찰자의 시점 변수(Camera Angle, Altitude, Lens 설정)를 판단하여 상태(State) 값으로 업데이트.
    *   **분석 결과 구조화 발동:** `ANALYZE` 기능 호출 즉시 `#.정보분석샘플` 포맷을 적용하여 아래 3개 섹션으로 분석 결과를 구조화하여 출력.
    *   옵션 패널(Right Sidebar)이 자동으로 열리며 분석 중 텍스트가 표시됨.

> **[분석 결과 출력 포맷 — `#.정보분석샘플`]**
>
> **섹션 1. 광학 및 시점 파라미터 (Optical & Viewpoint Data)**
>
> 입력된 이미지의 카메라 위치 및 촬영 환경을 수치화된 데이터로 변환한 값입니다.
>
> | 변수 (Variable) | 구조화 데이터 (Value) |
> | :--- | :--- |
> | **촬영 시점 (Viewpoint)** | — |
> | **방위각 (Azimuth)** | — |
> | **촬영 고도 (Altitude)** | — |
> | **투시 왜곡 (Perspective)** | — |
> | **센서 포맷 (Sensor)** | — |
> | **초점 거리 (Focal Length)** | — |
> | **광원 및 날씨 (Lighting)** | — |
> | **대비 강도 (Contrast)** | — |
>
> ---
>
> **섹션 2. 기하학 및 공간 구조 명세 (Geometric & Spatial Specifications)**
>
> 건축물의 매스와 파사드 구성을 층위별 시스템 구조로 분해한 데이터입니다.
>
> | 시스템 분류 | 구조 및 형태 사양 |
> | :--- | :--- |
> | **외피 시스템 (Skin)** | — |
> | **내부 파사드 (Inner)** | — |
> | **외부 파사드 (Outer)** | — |
> | **기본 매스 (Mass)** | — |
> | **하층부 (1F Base)** | — |
> | **중층부 (Mid Body)** | — |
> | **상층부 (Roof)** | — |
>
> ---
>
> **섹션 3. 개념 및 시각적 속성 (Conceptual & Visual Attributes)**
>
> 디자인 방법론과 감성적 품질을 측정 가능한 시각적 키워드 세트로 추출한 결과입니다.
>
> | 속성 분류 | 데이터 지표 |
> | :--- | :--- |
> | **디자인 알고리즘** | — |
> | **주조색 (Color Palette)** | — |
> | **형태 모티브 (Motif)** | — |
> | **형태적 대비 (Form)** | — |
> | **감성적 대비 (Mood)** | — |

---

## **PHASE 3. VIEWPOINT CONFIGURATION (뷰 선택 및 이미지 생성)**
사용자가 Generate 버튼을 클릭하는 순간. PHASE 1에서 확정된 분석 데이터를 기반으로, 선택된 뷰 설정에 따른 프롬프트를 발동하여 이미지를 생성하는 단계.

1.  **Action Trigger**: 사용자가 우측 하단의 [Generate] 버튼 클릭.

2.  **View Selection (뷰 선택)**: 사용자가 다음 세 가지 뷰 옵션 중 하나를 선택.

| 뷰 이름 | Azimuth | Altitude | Lens |
|---|---|---|---|
| **아이레벨투시뷰** | 04:30 또는 07:30 | 1.6m | 24-32mm |
| **정면뷰** | 06:00 | 사용자 직접 입력 | 50mm |
| **탑뷰** | 06:00 | 300m | 24mm |

3.  **View Prompt Activation (선택 뷰 프롬프트 자동 발동)**: 선택된 뷰에 따라 아래 해당 프롬프트가 자동 발동됨.

---

### **[View A] 아이레벨투시뷰**

<마크다운>

# GOAL
* Generate a precise "Eye-Level Corner View" of the architecture presented in the source image.
* Treat the building in the image as a completed, constructed reality.
* Change the angle of view to this specific new perspective without altering the building's original geometry, materials, or style.
* Render the simulation strictly by resetting the **Angle of View** and the **Optical Environment**.

# **Photographer's workflow**
**Manual Entry: Capturing a Dynamic Eye-Level Corner View**

This professional technique utilizes a **two-point perspective** to maximize a building's sense of form, volume, and depth.

**Method:**
1.  **Positioning:** Place the camera at ground level, maintaining a standard **eye-level height (approx. 1.5 - 1.8 meters)**. Frame the shot from a corner, allowing two facades of the building to recede into the frame.

2.  **Photographic Focus:** The objective is to capture the interplay between the building's primary surfaces and its structural rhythm. Concentrate on how the light models the **volumetric mass** and reveals the **texture of the main facade materials**. The pattern of fenestration (windows) should be used to create a sense of scale and repetition.

3.  **Lighting:** As a standard professional practice, shoot under the **soft, even, diffused light of an overcast day**. This neutral lighting is chosen to avoid harsh shadows that can obscure architectural forms, allowing for a clear and objective documentation of the materials and design.

4.  **Lens and Perspective Control:** Use a **wide-angle lens (e.g., 24-35mm)** to create a sense of presence. It is critical to use a **tilt-shift lens** or apply meticulous perspective correction in post-production. All vertical lines must be rendered perfectly straight to maintain architectural integrity.

# Specification: Eye-Level Corner View
* **Shooting Intent**: Naturally expresses the building's three-dimensionality and surrounding context, and provides a sense of stability by perfectly controlling vertical distortion.
* **Camera**: Sony A7R V (61MP High Resolution)
* **Lens**: Canon TS-E 24mm f/3.5L II (Tilt-Shift Lens, using adapter)
* **Aperture**: f/11 (Securing sharpness across the entire image with pan focus)
* **ISO**: 100 (Best image quality and minimum noise)
* **Shutter Speed**: 1/125 sec
* **Other Equipment**: Sturdy Tripod (Prevention of shake and long exposure)

---
<json code>
{
  "prompt": {
    "goal": [
      "Generate a precise, high-resolution, photorealistic 'Eye-Level Corner View' of the architecture from the source image.",
      "Strictly preserve the building's original geometry, volume, material textures, and architectural style as a complete, constructed reality.",
      "Simulate a seamless angle-of-view change to this new ground-level perspective without introducing form distortion.",
      "Render the simulation by strictly resetting the Angle of View and the Optical Environment to professional standards."
    ],
    "photographers_workflow": {
      "title": "Manual Entry: Capturing a Dynamic, Corrected Eye-Level Corner View",
      "description": "This professional technique utilizes a perfect two-point perspective to maximize a building's sense of form, volume, and depth, while ensuring geometric accuracy.",
      "method": {
        "positioning": "Place the camera at street level, maintaining a standard eye-level height (approx. 1.7 meters). Position the camera at a corner intersection to allow two primary facades to recede into the frame with balanced prominence.",
        "photographic_focus": "The core objective is a clear, objective documentation of the architecture. Concentrate on the interplay of form and material textures. The pattern of fenestration and structural rhythm must be sharp and precise to establish a clear sense of scale and repetition.",
        "lighting": "Shoot under the heavily diffused, even light of a high-level overcast cloud cover. This neutral, soft lighting is essential to reveal the full textural details of all materials without harsh shadows or specular glare, allowing for an accurate architectural representation.",
        "lens_and_perspective_control": "Use a wide-angle perspective control (tilt-shift) lens, equivalent to 24mm on a full-frame sensor. It is critical to meticulously apply shift controls (and perspective correction in post-production if needed) to ensure all vertical lines are perfectly straight and parallel to the frame, maintaining architectural integrity and stability."
      }
    },
    "specification": {
      "view_type": "Corrected Eye-Level Corner View",
      "details": {
        "shooting_intent": "Naturally and accurately expresses the building's three-dimensionality and surrounding urban context, providing a sense of monumental presence and stability by perfectly controlling vertical distortion.",
        "camera": "Full-Frame Mirrorless (High-Resolution Sensor, e.g., Sony A7R V or equivalent)",
        "lens": "24mm f/3.5 Perspective Control (Tilt-Shift) Lens",
        "aperture": "f/11 (To ensure edge-to-edge sharpness with pan focus across the architectural features)",
        "iso": 100,
        "shutter_speed": "1/160 sec",
        "other_equipment": "Gitzo Systematic Tripod with Geared Head (For precise alignment and stabilization), Cable Release"
      }
    }
  }
}

---

### **[View B] 정면뷰**

<마크다운>

# GOAL
* Generate a precise "Front Elevation View" of the architecture presented in the source image.
* Treat the building in the image as a completed, constructed reality.
* Change the angle of view to this specific new perspective without altering the building's original geometry, materials, or style.
* Render the simulation strictly by resetting the **Angle of View** and the **Optical Environment**.

# PHOTOGRAPHER'S WORKFLOW
**Manual Entry: Capturing a Distortion-Free Elevation (Front) View**
This professional technique is used to create a perfectly flat, perspective-corrected representation of a building's facade, applicable to both front and side views.

**Method:**
1.  **Positioning:** Place the camera directly facing the facade, ensuring the camera's sensor plane is perfectly parallel to the building's surface.
2.  **Photographic Focus:** The objective is to document the formal composition of the facade. Concentrate on the building's sense of proportion, the rhythm of its modular elements (such as windows or panels), and its pure materiality. The goal is to represent the architect's two-dimensional design intent with absolute clarity.
3.  **Lighting:** As a standard professional practice, shoot under the soft, even, diffused light of an overcast day. This neutral lighting is chosen to render textures and colors accurately without shadows that could obscure the facade's details or its flatness.
4.  **Lens and Perspective Control:** The use of a tilt-shift lens is non-negotiable for this type of shot. It allows for precise composition while keeping the camera level, thus rendering all vertical and horizontal lines perfectly parallel to the frame. A telephoto lens can also be used from a distance to further compress the perspective.

# SPECIFICATION: FRONT ELEVATION VIEW
* **Shooting Intent:** Records the architect's design intent accurately and objectively like a 2D drawing. Minimizes perspective to emphasize the proportion and rhythm of the facade.
* **Camera:** Sony A7R V
* **Lens:** Canon TS-E 50mm f/2.8L MACRO (Standard angle Tilt-Shift Lens)
* **Aperture:** f/11
* **ISO:** 100
* **Shutter Speed:** 1/125 sec
* **Other Equipment:** Tripod

---
<json code>
{
  "prompt_configuration": {
    "goal": [
      "Generate a precise 'Front Elevation View' of the architecture presented in the source image.",
      "Treat the building in the image as a completed, constructed reality.",
      "Change the angle of view to this specific new perspective without altering the building's original geometry, materials, or style.",
      "Render the simulation strictly by resetting the Angle of View and the Optical Environment."
    ],
    "photographers_workflow": {
      "technique": "Manual Entry: Capturing a Distortion-Free Elevation (Front) View",
      "description": "This professional technique is used to create a perfectly flat, perspective-corrected representation of a building's facade, applicable to both front and side views.",
      "method": {
        "positioning": "Place the camera directly facing the facade, ensuring the camera's sensor plane is perfectly parallel to the building's surface.",
        "photographic_focus": "The objective is to document the formal composition of the facade. Concentrate on the building's sense of proportion, the rhythm of its modular elements (such as windows or panels), and its pure materiality. The goal is to represent the architect's two-dimensional design intent with absolute clarity.",
        "lighting": "As a standard professional practice, shoot under the soft, even, diffused light of an overcast day. This neutral lighting is chosen to render textures and colors accurately without shadows that could obscure the facade's details or its flatness.",
        "lens_and_perspective_control": "The use of a tilt-shift lens is non-negotiable for this type of shot. It allows for precise composition while keeping the camera level, thus rendering all vertical and horizontal lines perfectly parallel to the frame. A telephoto lens can also be used from a distance to further compress the perspective."
      }
    },
    "specification": {
      "view_type": "FRONT ELEVATION VIEW",
      "shooting_intent": "Records the architect's design intent accurately and objectively like a 2D drawing. Minimizes perspective to emphasize the proportion and rhythm of the facade.",
      "equipment_and_settings": {
        "camera": "Sony A7R V",
        "lens": "Canon TS-E 50mm f/2.8L MACRO (Standard angle Tilt-Shift Lens)",
        "aperture": "f/11",
        "iso": 100,
        "shutter_speed": "1/125 sec",
        "other_equipment": "Tripod"
      }
    }
  }
}

---

### **[View C] 탑뷰**

<마크다운>

# GOAL
* Generate a precise "Orthographic TOP View" of the architecture presented in the source image.
* Treat the building in the image as a completed, constructed reality.
* Change the angle of view to this specific new perspective without altering the building's original geometry, materials, or style.
* Render the simulation strictly by resetting the **Angle of View** and the **Optical Environment**.

# **Photographer's workflow**
**Manual Entry: Capturing the Orthographic Top View**

This professional technique is used to create a perfectly flat, non-perspective, two-dimensional image of a building's roof, often referred to as a plan view.

**Method:**
1.  **Positioning:** Place the camera (typically a drone) at a sufficient altitude, positioned directly and vertically above the center of the building.

2.  **Photographic Focus:** The objective is to capture the **graphic composition of the roof plane**. The focus should be on the building's overall **footprint, the geometric relationship between its forms and voids (such as courtyards or terraces), and the patterns created by roofing materials and structures**. The goal is to produce a clear, diagram-like representation of the building from above.

3.  **Lighting:** As a standard professional practice, shoot under the **soft, even, diffused light of an overcast day**. This is critical to eliminate all shadows, which allows the pure geometry and texture of the roof plan to be documented with maximum clarity.

4.  **Technique and Perspective Control:** This view requires a true **orthographic projection**. This is achieved through drone-based photogrammetry processing or a direct orthographic render. All sense of perspective must be eliminated, resulting in a perfect two-dimensional image. Frame the composition in a **1:1 square aspect ratio** to reinforce the plan-like quality.

### Specification: Top View
* **Shooting Intent**: Shows the roof plan configuration accurately without distortion and graphically expresses the relationship with the surrounding site.
* **Camera**: DJI Mavic 3 Pro Cine (Drone)
* **Lens**: 24mm Hasselblad Camera (Main Wide-angle Camera)
* **Aperture**: f/8
* **ISO**: 100
* **Shutter Speed**: 1/250 sec
* **Other Equipment**: Vertical Descent Shooting Mode (Securing accurate vertical view)

---
<json code>
{
  "prompt": {
    "goal": [
      "Generate a precise \"Orthographic TOP View\" of the architecture presented in the source image.",
      "Treat the building in the image as a completed, constructed reality.",
      "Change the angle of view to this specific new perspective without altering the building's original geometry, materials, or style.",
      "Render the simulation strictly by resetting the Angle of View and the Optical Environment."
    ],
    "photographers_workflow": {
      "title": "Manual Entry: Capturing the Orthographic Top View",
      "description": "This professional technique is used to create a perfectly flat, non-perspective, two-dimensional image of a building's roof, often referred to as a plan view.",
      "method": {
        "positioning": "Place the camera (typically a drone) at a sufficient altitude, positioned directly and vertically above the center of the building.",
        "photographic_focus": "The objective is to capture the graphic composition of the roof plane. The focus should be on the building's overall footprint, the geometric relationship between its forms and voids (such as courtyards or terraces), and the patterns created by roofing materials and structures. The goal is to produce a clear, diagram-like representation of the building from above.",
        "lighting": "As a standard professional practice, shoot under the soft, even, diffused light of an overcast day. This is critical to eliminate all shadows, which allows the pure geometry and texture of the roof plan to be documented with maximum clarity.",
        "technique_and_perspective_control": "This view requires a true orthographic projection. This is achieved through drone-based photogrammetry processing or a direct orthographic render. All sense of perspective must be eliminated, resulting in a perfect two-dimensional image. Frame the composition in a 1:1 square aspect ratio to reinforce the plan-like quality."
      }
    },
    "specification": {
      "view_type": "Top View",
      "details": {
        "shooting_intent": "Shows the roof plan configuration accurately without distortion and graphically expresses the relationship with the surrounding site.",
        "camera": "DJI Mavic 3 Pro Cine (Drone)",
        "lens": "24mm Hasselblad Camera (Main Wide-angle Camera)",
        "aperture": "f/8",
        "iso": 100,
        "shutter_speed": "1/250 sec",
        "other_equipment": "Vertical Descent Shooting Mode (Securing accurate vertical view)"
      }
    }
  }
}

---

4.  **Image Generation (AI 렌더링 실행)**:
    *   **Engine:** `gemini-3.1-flash-image-preview` (MODEL_IMAGE_GEN)
    *   **입력 데이터:** 원본 이미지 + PHASE 1 분석 결과(`#.정보분석샘플` 포맷 3개 섹션) + 선택된 뷰 프롬프트(Markdown + JSON)
    *   **실행 차단 조건:** 업로드 이미지가 없거나 뷰 선택이 완료되기 전 실행 불가.
    *   **Compliance Check**:
        *   [ ] 원본 형태 100% 보존? (No Hallucination)
        *   [ ] 퍼스펙티브 수학적 정확성? (No Distortion)
        *   [ ] 비가시권 논리적 완성? (No Blank Spaces)
        *   [ ] **정면(06:00) 방위 보존?** (No Reorientation)

5.  **Result Projection (결과 출력 및 캔버스 주입)**:
    *   생성된 이미지를 `simulationViewpoint` 상태에 저장.
    *   캔버스에 원본 이미지 우측에 새 `CanvasItem`으로 주입. 선택 자동 활성화.
    *   다운로드 버튼 활성화 (`simulation.png`).