# Temporal Masking Listening Test

A two-stage adaptive psychoacoustic listening test for investigating temporal masking.

## Repository structure

```
masking-test/
├── index.html          ← The test application
├── config.json         ← All study parameters (edit this, not the HTML)
├── README.md
└── stimuli/
    ├── simultaneous_0ms_1kHz_20dB.wav
    ├── simultaneous_0ms_1kHz_25dB.wav
    ├── ...
    ├── forward_10ms_1kHz_55dB.wav
    ├── backward_20ms_2kHz_60dB.wav
    └── ...
```

---

## Filename convention

All audio files must follow this exact pattern:

```
{condition}_{delay}_{frequency}_{level}dB.wav
```

| Part        | Examples                          | Notes                            |
|-------------|-----------------------------------|----------------------------------|
| condition   | `simultaneous`, `forward`, `backward` | simultaneous = Stage 1 calibration |
| delay       | `0ms`, `10ms`, `20ms`, `50ms`     | Always include ms suffix         |
| frequency   | `1kHz`, `2kHz`                    | Must match config.json exactly   |
| level       | `20dB`, `55dB`, `80dB`            | Always include dB suffix         |

### Examples

```
# Stage 1 — calibration (simultaneous masking, no delay)
simultaneous_0ms_1kHz_20dB.wav
simultaneous_0ms_1kHz_25dB.wav
simultaneous_0ms_1kHz_30dB.wav
simultaneous_0ms_1kHz_35dB.wav
simultaneous_0ms_1kHz_40dB.wav
simultaneous_0ms_1kHz_45dB.wav
simultaneous_0ms_1kHz_50dB.wav
simultaneous_0ms_1kHz_55dB.wav
simultaneous_0ms_1kHz_60dB.wav
simultaneous_0ms_1kHz_65dB.wav
simultaneous_0ms_1kHz_70dB.wav
simultaneous_0ms_1kHz_75dB.wav
simultaneous_0ms_1kHz_80dB.wav
# (repeat above for 2kHz)

# Stage 2 — forward masking (noise before probe)
forward_10ms_1kHz_50dB.wav
forward_10ms_1kHz_55dB.wav
forward_20ms_1kHz_55dB.wav
forward_50ms_1kHz_60dB.wav
# (repeat for 2kHz, other delays, other levels)

# Stage 2 — backward masking (noise after probe)
backward_10ms_1kHz_55dB.wav
backward_20ms_2kHz_60dB.wav
```

---

## config.json reference

```json
{
  "researcher": {
    "emailjs_key":      "YOUR_EMAILJS_PUBLIC_KEY",
    "emailjs_service":  "YOUR_SERVICE_ID",
    "emailjs_template": "YOUR_TEMPLATE_ID",
    "researcher_email": "you@university.ac.uk"
  },

  "staircase": {
    "rule":                    "2down1up",   // convergence rule
    "start_level":             70,           // starting probe level (dB)
    "step_large":              10,           // initial step size (dB)
    "step_small":              5,            // step after first reversal (dB)
    "reversals_total":         8,            // stop staircase after this many reversals
    "reversals_for_threshold": 6,            // use last N reversals to compute threshold
    "min_level":               20,           // floor level (dB)
    "max_level":               80            // ceiling level (dB)
  },

  "stimuli": {
    "path":              "stimuli/",         // folder containing audio files
    "frequencies":       ["1kHz", "2kHz"],   // test frequencies (order will be randomised)
    "level_step":        5,                  // step size between level files (dB) — informational
    "masker_level":      80,                 // masker noise level (dB) — informational
    "calibration_tag":   "simultaneous",     // condition name used for Stage 1 files
    "stage2_window":     10,                 // ±dB around threshold to select Stage 2 files
    "filename_pattern":  "{condition}_{delay}_{frequency}_{level}dB.wav"
  }
}
```

### Key parameters to adjust

- **`stage2_window`** — increase if too few Stage 2 trials are being selected, decrease for tighter control
- **`reversals_total`** / **`reversals_for_threshold`** — standard is 8 total, mean of last 6
- **`step_large`** / **`step_small`** — 10/5 dB is standard; use 6/2 for finer resolution
- **`start_level`** — set comfortably above expected threshold so the staircase descends first

---

## EmailJS setup (one-time, ~5 minutes)

1. Go to [emailjs.com](https://www.emailjs.com) and create a free account (200 emails/month free)
2. **Email Services** → Add New Service → connect Gmail, Outlook, or SMTP
3. **Email Templates** → Create Template. In the template body use:
   - `{{subject}}` — auto-set to "Masking Test — [name] — [date]"
   - `{{message}}` — participant summary (demographics + thresholds)
   - `{{csv_data}}` — full trial data as CSV text
   - Set **To** field to: `{{to_email}}`
4. **Account** → API Keys → copy your **Public Key**
5. Note your **Service ID** and **Template ID** from their respective pages
6. Paste all three into `config.json` under `"researcher"`, along with your email address

---

## GitHub Pages deployment

1. Create a GitHub account at [github.com](https://github.com)
2. Create a new **public** repository named `masking-test`
3. Upload `index.html`, `config.json`, and `README.md` to the root
4. Create the `stimuli/` folder: **Add file → Create new file** → type `stimuli/.gitkeep` → commit
5. Click into `stimuli/` → **Add file → Upload files** → drag all WAV files → commit
6. Go to **Settings → Pages → Source**: select `main` branch → Save
7. After ~60 seconds your test is live at: `https://yourusername.github.io/masking-test/`

### For large stimulus sets
Use [GitHub Desktop](https://desktop.github.com) — clone your repo, copy files into the `stimuli/` folder on your computer, then sync. Much faster than browser uploads for many files.

---

## Output CSV format

The downloaded/emailed CSV contains:
- Commented header lines (`#`) with all participant demographics and thresholds
- Full staircase log (level and response for every calibration trial)
- Stage 2 trial data: `trial, filename, condition, delay_ms, frequency, level_dB, heard_tone, timestamp`

---

## Generating stimuli in MATLAB

Suggested naming in MATLAB:
```matlab
conditions = {'simultaneous', 'forward', 'backward'};
delays_ms  = [0, 10, 20, 50, 100];
freqs      = {'1kHz', '2kHz'};
levels_dB  = 20:5:80;

for each combination:
  filename = sprintf('%s_%dms_%s_%ddB.wav', condition, delay, freq, level);
  audiowrite(fullfile('stimuli', filename), audio, fs);
```
