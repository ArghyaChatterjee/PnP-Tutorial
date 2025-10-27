# 📷 Perspective-n-Point (PnP) Tutorial

A detailed discussion on PnP.

## Introduction

Imagine you took an image of your house and showed it to your friend, and asked a simple question:

> “Can you tell me where this photograph is taken from?”

Will your friend be able to guess the spot?
If yes, then how?
How does our brain figure out the **location of the camera** just by looking at the image?

Well, this is a well-known problem in **Photogrammetry**, known as the **Perspective-n-Point (PnP)** problem.

---

## What Is PnP?

PnP is the problem of **estimating the relative pose of the camera** with respect to the environment,
given the **point correspondences** between the environment and the image.

---

### Let’s break it down

* **Pose** → Position and orientation of the camera in the environment.
* **Relative** → Because we need a reference coordinate system to define the camera’s pose.

  > Example: “The camera is 2 meters in front of the main door and inclined 45° upward from the ground.”
* **Point correspondences** → Which part of the 3D world corresponds to which part of the 2D image.

---

<div>![Point Correspondences between 3D Environment and Camera Image](media/pnp_correspondences.png)</div>

The figure above shows how an image of a 3D environment is captured by a camera.
Here, the 3D points **X₁, X₂, X₃** correspond to 2D image points **x₁, x₂, x₃**.

Finding these correspondences seems intuitive for humans, but it’s a **big challenge for computers**.

> 👉 *Click here to try an interactive version of this figure*

---

Basically, if we are given enough point correspondences, we can mathematically find the camera pose.
But how much is enough?

Math says: only **three perfect point correspondences** are enough to recover the camera pose.
Hence, **PnP → P3P** problem.

---

## Why Exactly 3 Points?

This has something to do with **Degrees of Freedom**,
which is well explained in the linked video in the original article.

---

## 🧮 Problem Formulation

Let’s discuss the case of **P3P** and properly define the problem.

**Given:**

* 3D points { X₁, X₂, X₃ }
* Their corresponding 2D image points { x₁, x₂, x₃ }
* Focal length of the camera *f*

**Find:**

* 3D translation vector of the camera (**t**)
* 3×3 rotation matrix of the camera (**R**)

> **Note:**
> We assume an *ideal pinhole camera* (no lens distortion)
> and the **principal point** is exactly at the image center.

If you’re unfamiliar with these concepts, see the author’s upcoming post on *Single View Geometry*.

---

## 🔢 Solution Steps

To simplify the process, let’s break it into **four main steps**:

1. Compute angles between projection rays
2. Compute lengths of projection rays
3. Identify the correct solution
4. Compute the pose

---

### 1️⃣ Compute Angles Between Projection Rays

**Given:** Image points { x₁, x₂, x₃ } and focal length *f*
**Find:** Angles between projection rays { α, β, γ }

<div>![Image Formation in Pinhole Camera Model](media/pinhole_model.png)</div>

Assume everything is defined in the **camera coordinate system**,
with the camera at **(0, 0, 0)** facing the +Z axis.

<div>![Coordinates in the Image Plane](media/image_plane_coords.png)</div>

---

### 2️⃣ Compute Lengths of Projection Rays

**Given:** Angles { α, β, γ } and 3D points { X₁, X₂, X₃ }
**Find:** Lengths { s₁, s₂, s₃ }

<div>![Tetrahedron Geometry](media/tetrahedron_geometry.png)</div>

This step deals with the **geometry of tetrahedron X₀–X₁–X₂–X₃**.

First, find relative distances between the 3D points { a, b, c },
using simple Euclidean distance.

<div>![Relative Distances Between 3D Points](media/distances_between_points.png)</div>

To find { s₁, s₂, s₃ }, apply the **Law of Cosines**.

For triangle X₀–X₁–X₂:

<div>![Law of Cosines Equation Illustration](media/law_of_cosines.png)</div>

Repeat the same for all three faces of the tetrahedron.

<div>![Equations of All Faces](media/tetrahedron_equations.png)</div>

Equation (vii) yields a **fourth-degree polynomial**.
Solving it gives **four possible values** for *v*,
which can be substituted into equation (vi) to obtain *u*.

Hence, we get **four sets of (u,v)** values — only one of which is correct.

---

### 3️⃣ Identify the Correct Solution

<div>![Four Possible Solutions](media/four_possible_solutions.png)</div>

Geometrically, it’s possible to get **four different sets** of { s₁, s₂, s₃ }
for the same angles { α, β, γ } and relative distances { a, b, c }.

**Methods to choose the correct solution:**

1. Use additional sensors like **GPS** or **IMU** to get an initial pose estimate
   and select the solution nearest to it.
2. Use **more correspondence points** — e.g., add a fourth pair (X₄, x₄).

   * Compute four camera poses using Step 4 below and check which aligns best with X₄.
   * Alternatively, solve for { s₁, s₂, s₄ }, { s₂, s₃, s₄ }, { s₁, s₃, s₄ }
     and find the set consistent with previous { s₁, s₂, s₃ } values.

---

### 4️⃣ Compute Pose

**Given:** Complete tetrahedron geometry
**Find:** Camera pose — rotation matrix R<sub>c</sub> and translation vector t<sub>c</sub>

<div>![Camera Pose Computation Diagram](media/pose_computation.png)</div>

We already have { X₁, X₂, X₃ } in both **camera** and **world** coordinate systems.
Now we need a **transformation from camera → world coordinates**,
which lets us transform the camera position to obtain the final **camera pose**.

This is a classic **3D registration** (or point-cloud alignment) problem,
also known as *scan matching*.

> The author will cover this topic in a future post.
> You can check related works referenced in the article for more ideas.

---

## 💭 Final Thoughts

This post explains one of the earliest variants of the **PnP algorithms**,
originally proposed by **Grunert in 1841**.

Centuries later, we have much faster and more robust solutions
(e.g., OpenCV’s `solvePnP`, PyPose implementations).
Still, understanding the **original geometric formulation**
helps you apply and debug modern methods more effectively.

---

## 📚 References

* *Multiple View Geometry in Computer Vision* — Richard Hartley & Andrew Zisserman
* Presentation and slides by **Prof. Cyrill Stachniss**
* Blog post by **Jingnan Shi**
* Lecture by **Prof. Steven LaValle**

