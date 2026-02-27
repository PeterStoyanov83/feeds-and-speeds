# ⚙️ CNC Feeds & Speeds Calculator

**A Stoyanov Workshop Tool**

> Metric feeds & speeds reference for the **Onefinity Woodworker CNC** — built for real workshop use, not theory.

---

## 🎸 About

This tool was built as part of the **Stoyanov Guitars** workshop workflow — a custom guitar building operation running an Onefinity Woodworker CNC with Buildbotics controller.

It covers everything from delicate **0.4mm micro bits** for fine detail inlay work, all the way through to **12.7mm (1/2") end mills** for body profiling — with chipload ranges calibrated for wood and wood-like materials.

**[→ stoyanov-guitars.com](https://www.stoyanov-guitars.com)**

---

## 🛠️ Features

### Calculator
- Live chipload calculation with instant in-range / too-high / too-low feedback
- Feed rate slider calibrated to Onefinity's actual range (50–12,700 mm/min)
- Spindle RPM up to 40,000
- 1–4 flute selection

### Tool Library — 16 sizes across 4 groups
| Group | Sizes | Use Case |
|-------|-------|----------|
| 🔴 **Micro** | 0.4 / 0.5 / 0.6 / 0.8 / 1.0 mm | Inlay detail, fine engraving |
| 🟡 **Small** | 2 / 3 / 3.175 (1/8") / 4 mm | Tight pockets, binding channels |
| 🟢 **Standard** | 5 / 6 / 6.35 (1/4") / 8 mm | General routing, body work |
| 🔵 **Large** | 10 / 12 / 12.7 (1/2") mm | Roughing, body profiling |

### Material Presets
- Hardwood
- Plywood
- MDF
- Soft Plastic
- Hard Plastic

### Visual Tabs
- **Chipload Explained** — animated top-down view of the bit rotating with chip thickness visualized live
- **V-Bit Depth** — cross-section diagram showing how channel width controls cut depth, with 60° and 90° bit presets
- **Pass Depth** — side view showing multi-pass strategy and ramp entry

---

## 📐 The Formulas

```
CHIPLOAD   = Feed Rate ÷ (RPM × Number of Flutes)
FEED RATE  = RPM × Flutes × Chipload
RPM        = Feed Rate ÷ (Flutes × Chipload)
PLUNGE     = Feed Rate ÷ 2
PASS DEPTH = 1× – 2× tool diameter (0.25×–0.5× for micro bits)
```

---

## 🚀 Usage

No installation. No build step. Just open `index.html` in any browser.

Loads React via CDN — requires internet on first open, then works from cache.

---

## ⚠️ Micro Bit Warning

Bits under 1mm require:
- Maximum RPM (30,000–40,000)
- Very low feed rate (100–300 mm/min)
- **Ramp entry only** — never straight plunge
- Extremely rigid workholding

One mistake = broken bit.

---

## 🎸 Stoyanov Guitars

Custom handbuilt electric guitars made in Bulgaria.
Precision CNC manufacturing combined with traditional luthiery.

**[www.stoyanov-guitars.com](https://www.stoyanov-guitars.com)**

---

*Built for the Onefinity Woodworker / Buildbotics controller. All values in metric (mm, mm/min).*