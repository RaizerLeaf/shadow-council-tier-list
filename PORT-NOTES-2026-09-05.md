# Port notes: changes after adding Stacy (5 September 2026)

Source: https://github.com/RaizerLeaf/shadow-council-tier-list (live at https://raizerleaf.github.io/shadow-council-tier-list/).
Base: the #THE-STANCE `index.html` template described in TIER-LIST-METHOD.md.

Everything below was made to `index.html` after the Stacy entry landed, in these commits:

| Commit | Change |
|---|---|
| aa2b2cd | Timed "New" badge driven by an `added` date on the entry |
| d63a6b9 | Rewrote the DLSS vs FSR tab as a feature-by-feature breakdown |
| d433291 | Added Intel XeSS as a third column |
| 0f80d01 | Renamed the tab button |

None of these touch the scoring formula, `TIERS`, `GPU_DATABASE`, `computeRankings` or the comparison engine. Scores and ranks are unchanged. Apply them to another list by pasting the snippets in sections 1 and 2; each one names the exact anchor in the template it replaces or sits next to.

---

## 1. Timed "New" badge

Same behaviour as the existing "Updated" badge: shows beside the name for `UPDATED_BADGE_DAYS` (60) after the date, then hides itself. Both badges can show at once. The two share one renderer.

### 1a. Field guide comment

Add above the `updated` line in the SYSTEMS field guide comment:

```js
//   added       ISO date (YYYY-MM-DD) the entry joined the list; shows a "New"
//               badge beside the name for UPDATED_BADGE_DAYS, then hides
//   updated     ISO date (YYYY-MM-DD) of the last hardware change; shows an
//               "Updated" badge beside the name for UPDATED_BADGE_DAYS, then hides
```

### 1b. CSS

Add directly above `.breakdown-note {`:

```css
/* "New" badge: same timed behaviour as "Updated", driven by the entry's added date */
.new-tag {
    font-size: 0.7rem;
    background-color: #1d4ed8;
    color: #dbeafe;
    padding: 0.1rem 0.4rem;
    border-radius: 4px;
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: 0.03em;
}
```

### 1c. JavaScript

Replace the block that starts with the `// "Updated" badge:` comment and contains `const UPDATED_BADGE_DAYS = 60;` and `function renderUpdatedBadge(s)` with:

```js
// Dated badges: "New" (from `added`) and "Updated" (from `updated`) each show beside
// the name for this many days after their date, then hide themselves.
const UPDATED_BADGE_DAYS = 60;

function renderDatedBadge(isoDate, cls, text, titlePrefix) {
    if (!isoDate) return '';
    const when = new Date(isoDate + 'T00:00:00');
    if (isNaN(when)) return '';
    const ageDays = (Date.now() - when.getTime()) / 86400000;
    if (ageDays < 0 || ageDays > UPDATED_BADGE_DAYS) return '';
    const label = when.toLocaleDateString('en-GB', { day: 'numeric', month: 'short', year: 'numeric' });
    return ` <span class="${cls}" title="${titlePrefix} ${label}">${text}</span>`;
}

function renderNewBadge(s) {
    return renderDatedBadge(s.added, 'new-tag', 'New', 'Added to the list');
}

function renderUpdatedBadge(s) {
    return renderDatedBadge(s.updated, 'updated-tag', 'Updated', 'Hardware updated');
}
```

### 1d. Render both badges

In `renderSystemRows`, the name cell line becomes:

```js
<td class="name-cell">${escapeHtml(s.name)}${badge}${renderNewBadge(s)}${renderUpdatedBadge(s)}</td>
```

### 1e. Usage

Add `added: "YYYY-MM-DD"` to any new entry, next to `updated`. Example from this list:

```js
{
    id: "stacy", name: "Stacy", deviceClass: "desktop",
    cpu: "Ryzen 5 3600", cpuIndex: 58, cpuTier: 1,
    gpu: "RX 6700 XT", gpuRef: "RX 6700 XT", upscaler: "fsr",
    vram: 12, memory: 32, memoryType: "DDR4", unified: false, platform: "am4-3000",
    added: "2026-09-05",
    ...
}
```

---

## 2. Upscaler tab: DLSS vs FSR vs XeSS

The second tab of the GPU database section is now a three-column feature breakdown. For each feature it gives the version that introduced it (with date), a plain-language description, and a per-line list of the card series that can run it. An at-a-glance table and a closing callout follow the columns.

### 2a. Tab button

The button in the `.gpu-tabs` row:

```html
<button class="gpu-tab-btn" id="btn-gpu-tech-scaling" onclick="switchGpuTab('gpu-tech-scaling')">DLSS vs FSR vs XeSS</button>
```

### 2b. CSS

Add directly above `/* Specs column: wrap only at field boundaries */`:

```css
/* DLSS vs FSR vs XeSS feature breakdown */
.fx-columns { display: grid; grid-template-columns: repeat(3, minmax(0, 1fr)); gap: 1rem; }
.fx-col { border-radius: 8px; padding: 1rem; min-width: 0; }
.fx-col-dlss { background-color: rgba(16, 185, 129, 0.05); border: 1px solid rgba(16, 185, 129, 0.15); }
.fx-col-fsr { background-color: rgba(239, 68, 68, 0.05); border: 1px solid rgba(239, 68, 68, 0.15); }
.fx-col-xess { background-color: rgba(59, 130, 246, 0.05); border: 1px solid rgba(59, 130, 246, 0.15); }
.fx-col-title { font-size: 1.05rem; font-weight: 700; margin-bottom: 0.25rem; }
.fx-col-sub { font-size: 0.8rem; color: var(--text-muted); margin-bottom: 1rem; padding-bottom: 0.75rem; border-bottom: 1px solid rgba(255, 255, 255, 0.06); }
.fx-block { margin-bottom: 1rem; padding-bottom: 1rem; border-bottom: 1px solid rgba(255, 255, 255, 0.06); }
.fx-block:last-child { margin-bottom: 0; padding-bottom: 0; border-bottom: none; }
.fx-block-muted { opacity: 0.8; }
.fx-name { font-weight: 700; font-size: 0.92rem; color: var(--text-color); }
.fx-meta { font-size: 0.76rem; color: #93c5fd; margin: 0.15rem 0 0.4rem 0; }
.fx-desc { font-size: 0.84rem; color: #cbd5e1; margin-bottom: 0.4rem; }
.fx-list { list-style: none; margin: 0; padding: 0; font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, monospace; font-size: 0.76rem; color: var(--text-color); }
.fx-list li { padding: 0.15rem 0 0.15rem 1rem; position: relative; }
.fx-list li::before { content: "\25B8"; position: absolute; left: 0; color: var(--text-muted); }
.fx-note { font-size: 0.76rem; color: var(--text-muted); margin-top: 0.4rem; }
.fx-matrix-wrap { overflow-x: auto; }
.fx-matrix { width: 100%; border-collapse: collapse; font-size: 0.8rem; table-layout: auto; }
.fx-matrix th, .fx-matrix td { padding: 0.5rem 0.6rem; text-align: left; border-bottom: 1px solid rgba(255, 255, 255, 0.06); vertical-align: top; }
.fx-matrix th { font-size: 0.7rem; text-transform: uppercase; letter-spacing: 0.05em; color: var(--text-muted); background: none; }
.fx-matrix td:first-child { font-weight: 600; white-space: nowrap; }
.fx-callout { margin-top: 1.25rem; padding: 0.9rem 1rem; border-radius: 8px; background-color: rgba(59, 130, 246, 0.05); border: 1px solid rgba(59, 130, 246, 0.15); font-size: 0.88rem; color: #cbd5e1; }
@media (max-width: 900px) {
    .fx-columns { grid-template-columns: 1fr; }
    .fx-matrix td:first-child { white-space: normal; }
}
```

### 2c. Tab HTML

Replace everything from the `<!-- Tab 2: ... -->` comment through the closing `</div>` of `#gpu-tech-scaling` with the block below. It sits inside `.gpu-db-container`, after the `#gpu-perf-lookup` div.

**Adapt the last paragraph.** The `.fx-callout` block names cards and machines that are on the Shadow Council list (RTX 2060 through 4080, RX 6700 XT, RX 6800 XT, PS5 Pro, MSI Claw 8 AI+). Rewrite that one paragraph for the builds on your list; everything above it is list-independent.

```html
<!-- Tab 2: DLSS vs FSR vs XeSS feature breakdown -->
<div id="gpu-tech-scaling" class="gpu-tab-content" style="display: none; line-height: 1.6; font-size: 0.95rem; color: #e2e8f0;">
    <h4 style="font-weight: 600; color: #3b82f6; margin: 0 0 0.5rem 0;">DLSS vs FSR vs XeSS: what each does and which cards get it</h4>
    <p style="margin-bottom: 1.25rem; color: var(--text-muted); font-size: 0.9rem;">
        All three are families of features, not single switches. Each block below names the feature, the version that introduced it, and the card series that can run it. The score's <strong>features</strong> multiplier follows the same ladder: Multi Frame Gen 1.04, Frame Gen 1.02, machine-learned upscaling 1.00, shader FSR only 0.98, no ML upscaling and no hardware RT 0.92.
    </p>

    <div class="fx-columns">
        <!-- DLSS column -->
        <div class="fx-col fx-col-dlss">
            <div class="fx-col-title" style="color: #10b981;">NVIDIA DLSS</div>
            <div class="fx-col-sub">Deep Learning Super Sampling · runs on Tensor Cores, so GeForce RTX only. GTX 10 and GTX 16 cards have no Tensor Cores and get no DLSS at all.</div>

            <div class="fx-block">
                <div class="fx-name">Super Resolution and DLAA</div>
                <div class="fx-meta">Introduced: DLSS 2 (March 2020). DLSS 1 (2018) was trained per game and is retired. Rebuilt on a transformer model in DLSS 4 (January 2025); second-generation transformer in DLSS 4.5 (January 2026). DLAA is the same model run at native resolution as anti-aliasing.</div>
                <div class="fx-desc">Renders at a lower internal resolution and reconstructs the output frame with an AI model and motion vectors. The biggest single performance win in the set.</div>
                <ul class="fx-list">
                    <li>RTX 20</li>
                    <li>RTX 30</li>
                    <li>RTX 40</li>
                    <li>RTX 50</li>
                </ul>
                <div class="fx-note">RTX 20 and 30 lack FP8 support, so the DLSS 4 and 4.5 transformer models cost them a few percent more than the older CNN model did.</div>
            </div>

            <div class="fx-block">
                <div class="fx-name">Ray Reconstruction</div>
                <div class="fx-meta">Introduced: DLSS 3.5 (September 2023). Transformer model in DLSS 4 (January 2025).</div>
                <div class="fx-desc">Replaces hand-tuned ray-tracing denoisers with an AI denoiser, cleaning up reflections, shadows and path-traced lighting. Despite the 3.5 name it does not need Frame Generation hardware.</div>
                <ul class="fx-list">
                    <li>RTX 20</li>
                    <li>RTX 30</li>
                    <li>RTX 40</li>
                    <li>RTX 50</li>
                </ul>
            </div>

            <div class="fx-block">
                <div class="fx-name">Frame Generation (2X)</div>
                <div class="fx-meta">Introduced: DLSS 3 (October 2022). Moved from the Optical Flow Accelerator to an AI model in DLSS 4 (January 2025), which lowered its VRAM cost.</div>
                <div class="fx-desc">Inserts one AI-generated frame between every two rendered frames. Doubles displayed frame rate but not responsiveness, so it works best when the base frame rate is already 50 to 60 fps or more.</div>
                <ul class="fx-list">
                    <li>RTX 40</li>
                    <li>RTX 50</li>
                </ul>
            </div>

            <div class="fx-block">
                <div class="fx-name">Multi Frame Generation (3X, 4X, 6X)</div>
                <div class="fx-meta">Introduced: DLSS 4 (January 2025) with 3X and 4X modes. DLSS 4.5 (spring 2026) added the 6X mode and Dynamic Multi Frame Generation, which picks the multiplier to match your monitor's refresh rate.</div>
                <div class="fx-desc">Generates up to five frames per rendered frame. This is why an RTX 50 card earns the 1.04 features multiplier over an RTX 40 card's 1.02.</div>
                <ul class="fx-list">
                    <li>RTX 50</li>
                </ul>
            </div>

            <div class="fx-block fx-block-muted">
                <div class="fx-name">Reflex (latency reduction)</div>
                <div class="fx-meta">Introduced: September 2020. Reflex 2 with Frame Warp: RTX 50 first, later RTX 40.</div>
                <div class="fx-desc">Not part of DLSS but always bundled with Frame Generation to claw back the latency it adds. Base Reflex works on GTX 900 and newer.</div>
            </div>
        </div>

        <!-- FSR column -->
        <div class="fx-col fx-col-fsr">
            <div class="fx-col-title" style="color: #ef4444;">AMD FSR</div>
            <div class="fx-col-sub">FidelityFX Super Resolution · open source. FSR 1 to 3.1 run as ordinary shader code on almost any GPU, including GeForce GTX and RTX and Intel Arc. FSR 4 and Redstone are machine-learned and need Radeon hardware with AI accelerators.</div>

            <div class="fx-block">
                <div class="fx-name">Spatial upscaling</div>
                <div class="fx-meta">Introduced: FSR 1 (June 2021).</div>
                <div class="fx-desc">A single-frame sharpening upscaler with no motion data. Cheap and universal, but soft and shimmery next to everything that followed. Still the fallback in games that only ship FSR 1.</div>
                <ul class="fx-list">
                    <li>Radeon RX 400 and newer, plus Ryzen integrated graphics</li>
                    <li>GeForce GTX 900, GTX 10, GTX 16</li>
                    <li>GeForce RTX 20, 30, 40, 50</li>
                    <li>Intel Arc and Intel integrated graphics</li>
                </ul>
            </div>

            <div class="fx-block">
                <div class="fx-name">Temporal upscaling (shader-based)</div>
                <div class="fx-meta">Introduced: FSR 2 (May 2022). Improved in FSR 3 (September 2023) and FSR 3.1 (June 2024), which fixed most of the ghosting and made the upscaler swappable with DLSS in the same game.</div>
                <div class="fx-desc">Uses motion vectors and previous frames like DLSS, but with hand-written maths instead of a neural network. Good at Quality mode, visibly worse than DLSS at Performance mode. This is what the <strong>fsr</strong> rating (0.98) on the ladder means.</div>
                <ul class="fx-list">
                    <li>Radeon RX 500 minimum, RX 5000 and newer recommended</li>
                    <li>GeForce GTX 10 minimum, GTX 16 recommended</li>
                    <li>GeForce RTX 20, 30, 40, 50 (though DLSS is the better choice on these)</li>
                    <li>Intel Arc</li>
                </ul>
            </div>

            <div class="fx-block">
                <div class="fx-name">Frame Generation (shader-based)</div>
                <div class="fx-meta">Introduced: FSR 3 (September 2023). Decoupled from the upscaler in FSR 3.1 (June 2024), so an RTX 20 or 30 card can run DLSS Super Resolution and FSR Frame Generation together.</div>
                <div class="fx-desc">Interpolates one frame between two rendered frames using async compute shaders. Needs a solid 60 fps base to look right. AMD Fluid Motion Frames (AFMF) is the driver-level version that works in any game on RX 6000 and newer, at lower quality.</div>
                <ul class="fx-list">
                    <li>Radeon RX 5000 minimum, RX 6000 and newer recommended</li>
                    <li>GeForce RTX 20 minimum, RTX 30 and newer recommended</li>
                    <li>GeForce GTX 10 and 16: runs but not recommended, the cards are too slow to hold the base frame rate</li>
                </ul>
            </div>

            <div class="fx-block">
                <div class="fx-name">Machine-learned upscaling</div>
                <div class="fx-meta">Introduced: FSR 4 (March 2025), RX 9000 only. FSR 4.1 (June 2026) extended it to RX 7000 with a model tuned for RDNA 3.</div>
                <div class="fx-desc">AMD's answer to DLSS Super Resolution: a neural network run on the AI accelerators inside RDNA 4 and RDNA 3 compute units. Image quality is close to the DLSS 3 CNN model and well ahead of FSR 3.1. Games built on FSR 3.1 can be upgraded to FSR 4 through the Adrenalin driver.</div>
                <ul class="fx-list">
                    <li>Radeon RX 9000</li>
                    <li>Radeon RX 7000 (FSR 4.1 and later)</li>
                    <li>Not available on RX 6000 and older, GeForce GTX or RTX, or Intel Arc; those cards fall back to FSR 3.1</li>
                </ul>
            </div>

            <div class="fx-block">
                <div class="fx-name">Redstone: ML Frame Generation, Ray Regeneration, Radiance Caching</div>
                <div class="fx-meta">Introduced: FSR Redstone (December 2025). Ray Regeneration is AMD's equivalent of Ray Reconstruction; Radiance Caching is a neural cache for ray-traced global illumination, with game integrations rolling out through 2026.</div>
                <div class="fx-desc">Moves frame generation and the ray-tracing denoiser onto neural networks. ML Frame Generation is a clear step up from the FSR 3.1 interpolator, and there is no multi-frame mode yet.</div>
                <ul class="fx-list">
                    <li>Radeon RX 9000</li>
                    <li>RX 7000 and older get the shader-based FSR 3.1 fallback for frame generation and no Ray Regeneration</li>
                </ul>
            </div>
        </div>

        <!-- XeSS column -->
        <div class="fx-col fx-col-xess">
            <div class="fx-col-title" style="color: #3b82f6;">INTEL XeSS</div>
            <div class="fx-col-sub">Xe Super Sampling · every feature is a neural network with two code paths. The <strong>XMX</strong> path runs on the matrix engines in Intel Arc and is the full-quality version. The <strong>DP4a</strong> path runs the same network as integer shader maths on any GPU with Shader Model 6.4, so GeForce GTX 10 and newer and Radeon RX 5000 and newer can use XeSS too, a little slower and slightly softer.</div>

            <div class="fx-block">
                <div class="fx-name">Super Resolution (XeSS-SR)</div>
                <div class="fx-meta">Introduced: XeSS 1.0 (October 2022, with the Arc A770 and A750). Quality and preset updates through XeSS 1.3 (April 2024), which added Ultra Performance and Native AA modes.</div>
                <div class="fx-desc">Machine-learned temporal upscaling in the same family as DLSS Super Resolution and FSR 4. On the XMX path it sits between FSR 3.1 and DLSS in image quality; on DP4a it is still usually cleaner than FSR 3.1, which makes it the best free upgrade for RX 6000 and GTX 16 owners in games that ship it.</div>
                <ul class="fx-list">
                    <li>XMX: Intel Arc A-series, Arc B-series, Arc graphics in Core Ultra laptops and handhelds (Meteor Lake, Lunar Lake, Arrow Lake, Panther Lake)</li>
                    <li>DP4a: GeForce GTX 10, GTX 16, RTX 20, 30, 40, 50</li>
                    <li>DP4a: Radeon RX 5000, 6000, 7000, 9000</li>
                    <li>DP4a: Intel Xe integrated graphics (11th gen and newer)</li>
                </ul>
            </div>

            <div class="fx-block">
                <div class="fx-name">Ray-tracing denoiser</div>
                <div class="fx-meta">Not offered as of September 2026.</div>
                <div class="fx-desc">Intel has no shipping equivalent of Ray Reconstruction or Ray Regeneration; games on Arc use their own denoisers.</div>
                <ul class="fx-list">
                    <li>None</li>
                </ul>
            </div>

            <div class="fx-block">
                <div class="fx-name">Frame Generation (XeSS-FG) and Low Latency (XeLL)</div>
                <div class="fx-meta">Introduced: XeSS 2 (December 2024, with the Arc B580), Intel Arc only. XeSS 2.1 (August 2025) added a DP4a path so Nvidia and AMD cards can run it too, with XeLL enabled whenever frame generation is on.</div>
                <div class="fx-desc">One AI-generated frame between two rendered frames, plus a Reflex-style latency mode. On Arc the interpolation runs on the XMX matrix engines; on Nvidia and AMD it runs as compute shaders, closer to FSR 3.1 Frame Generation in cost and look.</div>
                <ul class="fx-list">
                    <li>XMX: Intel Arc A-series, Arc B-series, Arc graphics in Core Ultra processors</li>
                    <li>DP4a: GeForce GTX 10, GTX 16, RTX 20, 30, 40, 50</li>
                    <li>DP4a: Radeon RX 5000, 6000, 7000, 9000</li>
                </ul>
            </div>

            <div class="fx-block">
                <div class="fx-name">Multi Frame Generation (3X, 4X)</div>
                <div class="fx-meta">Introduced: XeSS 3 (January 2026, with the Core Ultra Series 3 Panther Lake launch). Driver updates in January and February 2026 turned it on for every game that already supports XeSS 2 frame generation.</div>
                <div class="fx-desc">Up to three AI-generated frames per rendered frame. Intel hardware only so far; there is no DP4a path for the multi-frame mode. Intel's B-series discrete cards and Panther Lake handhelds are the parts that gain the 1.04 rating from this.</div>
                <ul class="fx-list">
                    <li>Intel Arc B-series (Battlemage)</li>
                    <li>Intel Arc A-series (Alchemist)</li>
                    <li>Arc graphics in Core Ultra processors (Meteor Lake, Lunar Lake, Arrow Lake-H, Panther Lake)</li>
                    <li>Not available on GeForce or Radeon cards</li>
                </ul>
            </div>
        </div>
    </div>

    <div class="fx-matrix-wrap">
        <div class="modal-section-title" style="margin-top: 1.5rem;">At a glance</div>
        <table class="fx-matrix">
            <thead>
                <tr><th>Feature</th><th>DLSS version · cards</th><th>FSR version · cards</th><th>XeSS version · cards</th></tr>
            </thead>
            <tbody>
                <tr><td>Spatial upscaling</td><td>Never offered</td><td>FSR 1 (2021) · anything</td><td>Never offered</td></tr>
                <tr><td>Temporal upscaling, shader</td><td>Never offered</td><td>FSR 2 (2022), 3.1 (2024) · RX 500+, GTX 10+, all RTX, Arc</td><td>Never offered</td></tr>
                <tr><td>Machine-learned upscaling</td><td>DLSS 2 (2020), transformer in DLSS 4 (2025), 4.5 (2026) · RTX 20, 30, 40, 50</td><td>FSR 4 (2025) · RX 9000; FSR 4.1 (2026) · RX 7000</td><td>XeSS 1.0 (2022) · Arc on XMX; GTX 10+, all RTX, RX 5000+ on DP4a</td></tr>
                <tr><td>Ray-tracing denoiser</td><td>Ray Reconstruction, DLSS 3.5 (2023) · RTX 20, 30, 40, 50</td><td>Ray Regeneration, Redstone (2025) · RX 9000</td><td>Not offered</td></tr>
                <tr><td>Frame generation, shader</td><td>Never offered</td><td>FSR 3 (2023), 3.1 (2024) · RX 5000+, RTX 20+</td><td>XeSS 2.1 (2025) DP4a path · GTX 10+, all RTX, RX 5000+</td></tr>
                <tr><td>Frame generation, ML</td><td>DLSS 3 (2022) · RTX 40, 50</td><td>Redstone (2025) · RX 9000</td><td>XeSS 2 (2024) · Arc A, Arc B, Core Ultra Arc</td></tr>
                <tr><td>Multi frame generation</td><td>DLSS 4 (2025) 3X/4X, DLSS 4.5 (2026) 6X · RTX 50</td><td>Not yet</td><td>XeSS 3 (2026) 3X/4X · Arc A, Arc B, Core Ultra Arc</td></tr>
            </tbody>
        </table>
    </div>

    <div class="fx-callout">
        <strong>What this means for the builds on this list.</strong>
        Every RTX card here (2060 through 4080) gets DLSS Super Resolution and Ray Reconstruction, and the RTX 40 cards add Frame Generation. The RDNA 2 Radeons (RX 6700 XT, RX 6800 XT) and the consoles are on shader FSR 3.1: solid upscaling at Quality mode, shader frame generation, no ML denoiser. Those same Radeons can run XeSS Super Resolution on the DP4a path in games that ship it, which is usually the cleaner choice over FSR 3.1. The PS5 Pro is the exception with Sony's own PSSR neural upscaler, which is why it carries the <strong>ml</strong> rating. The MSI Claw 8 AI+ is the one XeSS-native machine on the ladder: its Arc 140V runs XeSS on the XMX path, including XeSS 3 multi-frame generation. An RTX 20 or 30 owner who wants frame generation can already have it today by pairing DLSS upscaling with FSR 3.1 or XeSS 2.1 Frame Generation in games that ship either one.
    </div>
</div>
```

### 2d. Facts the tab relies on (verified 5 September 2026)

- DLSS 4.5 (CES, January 2026): second-generation transformer Super Resolution on all RTX; 6X and Dynamic Multi Frame Generation on RTX 50 only, released spring 2026.
- FSR SDK 2.3 (June 2026): FSR Upscaling 4.1 brings machine-learned upscaling to RX 7000. ML Frame Generation 4.0 and Ray Regeneration stay RX 9000 only; older cards fall back to the FSR 3.1 shader paths.
- FSR Redstone launched December 2025 (ML Frame Generation, Ray Regeneration, Radiance Caching), RX 9000 only.
- XeSS 2.1 SDK (August 2025): XeSS Frame Generation and XeLL on non-Intel GPUs via DP4a (Shader Model 6.4: GTX 10 and newer, RX 5000 and newer).
- XeSS 3 (January 2026): Multi Frame Generation 3X/4X on Arc A-series, B-series and Core Ultra integrated Arc; no DP4a path.

Sources:
- https://www.nvidia.com/en-us/geforce/news/dlss-4-5-dynamic-multi-frame-gen-6x-2nd-gen-transformer-super-res/
- https://gpuopen.com/learn/amd-fsr-sdk-2-3-blog/
- https://gpuopen.com/amd-fsr-framegeneration/
- https://videocardz.com/newz/amd-fsr-redstone-launched-ml-based-upscaling-frame-gen-and-ray-regeneration-for-radeon-rx-9000-series
- https://www.tomshardware.com/pc-components/gpus/xess-sdk-2-1-release-opens-up-intels-framegen-tech-to-compatible-amd-and-nvidia-gpus-xe-low-latency-also-goes-cross-platform-if-framegen-is-enabled
- https://www.tomshardware.com/pc-components/gpu-drivers/intel-enables-xess-3-multi-frame-generation-in-latest-drivers-expanding-frame-generation-across-arc-gpus-and-core-ultra-igpus-mfg-can-be-enabled-across-any-title-with-xess-2-support

---

## 3. Noticed but not changed (candidates for TIER-LIST-METHOD.md)

These follow from the facts in 2d and affect the `upscaler` field, so they are flagged rather than applied.

- **RX 7000 cards now have ML upscaling.** With FSR 4.1 on RDNA 3, a Radeon RX 7000 build arguably rates `ml` (1.00) instead of `fsr` (0.98). The method doc's upscaler table still lists RDNA 3 under `fsr`. No RX 7000 card is on the Shadow Council list, so nothing moved here.
- **Intel Arc now has multi-frame generation.** XeSS 3 gives Arc A, Arc B and Core Ultra integrated Arc a 3X/4X mode. A strict reading of the features table would rate the MSI Claw 8 AI+ (Arc 140V) as `mfg` (1.04), lifting it from 19 to about 20. Left at `ml` because the doc ties `mfg` to DLSS 4 and the Claw's power envelope makes 4X of limited use. Decide once for both lists.

---

## 4. Verifying after a port

Run this from the repo root to print the ladder and confirm every `gpuRef` resolves:

```js
const fs = require('fs');
const html = fs.readFileSync('index.html', 'utf8');
const script = html.slice(html.indexOf('<script>') + 8, html.lastIndexOf('</script>'));
const src = script.replace(/document\.addEventListener\('DOMContentLoaded'[\s\S]*?\n    \}\);\n/, '');
const ctx = {};
new Function('module', src + '\n; module.out = { SYSTEMS, computeRankings, getGpuPerf, TIERS, renderNewBadge, renderUpdatedBadge };')(ctx);
const { SYSTEMS, computeRankings, getGpuPerf, TIERS, renderNewBadge, renderUpdatedBadge } = ctx.out;
console.log('unresolved gpuRefs:', SYSTEMS.filter(s => getGpuPerf(s.gpuRef) === 0).map(s => s.id));
let tier = null;
for (const s of computeRankings(SYSTEMS)) {
  if (s.tier !== tier) { tier = s.tier; console.log('\n== ' + TIERS.find(t => t.key === tier).label); }
  console.log(String(s.rank + (s.tied ? '=' : '')).padStart(4), String(s.score).padStart(3), s.name.padEnd(20),
    (renderNewBadge(s) ? 'NEW ' : '') + (renderUpdatedBadge(s) ? 'UPDATED' : ''));
}
```
