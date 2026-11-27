# Monitors

**5K at 27 inches** and **6K at 32 inches** hit the pixel density (~216–218 PPI) needed to run **clean 2x integer scaling** while keeping UI elements and text at a comfortable physical size for desktop viewing.

### The Core Scaling Trade-Off

Desktop operating systems were historically designed around an effective UI density of roughly **96 to 110 PPI** (pixels per inch). Modern high-DPI displays aim for ~2x that density so a single logical UI point maps cleanly to a **2 × 2 block of physical pixels** (integer scaling), avoiding sub-pixel blurring or extra GPU resampling overhead.

* **Why 4K falls short at 27" or 32":** A 27" 4K monitor (~163 PPI) set to 2x scaling produces a logical canvas of **1920 × 1080** (~82 logical PPI), making text and interface controls oversized. To get a comfortable UI size or workspace, users typically resort to **fractional scaling** (such as 125% or 150% on Windows, or "looks like" scaled modes in macOS).
* **Why 5K@27" works:** At **5120 × 2880** on 27" (~218 PPI), 2x scaling gives a **2560 × 1440** logical workspace (~109 logical PPI)—matching the familiar physical element size of a classic 27" 1440p monitor with razor-sharp rendering.
* **Why 6K@32" works:** At **6016 × 3384** on 32" (~216 PPI), 2x scaling gives a **3008 × 1692** logical workspace (~108 logical PPI)—delivering the same sharp pixel density and comfortable UI scale as 5K@27", but over a larger screen area.

---

### Resolution & Scaling Comparison

| Display Spec | Native PPI | 2x Integer Logical Resolution | Effective Logical PPI | Typical Result |
| --- | --- | --- | --- | --- |
| **27" 1440p** (2560×1440) | ~109 PPI | 2560×1440 (at 1x) | ~109 PPI | Standard UI size at 1x; visible pixels up close |
| **27" 4K** (3840×2160) | ~163 PPI | 1920×1080 (at 2x) | ~82 PPI | 2x is too large; often pushed to fractional scaling |
| **27" 5K** (5120×2880) | ~218 PPI | **2560×1440** (at 2x) | **~109 PPI** | Crisp HiDPI at comfortable 27" UI scale |
| **32" 4K** (3840×2160) | ~138 PPI | 1920×1080 (at 2x) | ~69 PPI | 2x is far too large; fractional scaling is common |
| **32" 6K** (6016×3384) | ~216 PPI | **3008×1692** (at 2x) | **~108 PPI** | Same sharp density as 5K@27" with more room |

### Practical Nuances

* **Fractional scaling isn't always broken:** Modern OS compositors handle fractional scaling far better than they used to, so using 125% or 150% on a 4K monitor is very common and usable in day-to-day work. The "requirement" refers to **pure 2x integer scaling without fractional compromises** at standard desktop distances.
* **Viewing distance and eyesight matter:** If you sit farther back or prefer larger UI elements, a lower-PPI display at 2x scaling can still feel natural; if you sit closer or want maximum text sharpness at ~100–110 logical PPI, ~215–220 PPI is the target.