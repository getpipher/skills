---
name: design-analyze
description: Analyze any website's UI/UX and generate AI-friendly prompts (read-only, safe anywhere)
argument-hint: "<website-url>"
allowed-tools: ["bash", "read", "write", "edit"]
---

# Design Analysis & Prompt Generator

**Purpose:** Extract design patterns from any website and translate them into reusable AI builder prompts.

**Scope:** User-level (works in any directory, any repo state)

**Safety:** ✅ Read-only operation, no file modifications, no git commands, no bash execution

**Usage:** `/design:analyze <website-url>`

---

## How It Works

### Input:
```
/design:analyze https://example.com
```

### Process:
1. **Fetch the website** (read-only, external)
2. **Analyze design elements** (no local modifications)
3. **Generate AI prompts** (output only)
4. **Suggest improvements** (recommendations only)

### Output:
- Design analysis summary
- Color palette (exact hex codes)
- Typography specifications
- Layout patterns
- AI-friendly prompts (short/medium/full)
- Improvement suggestions
- Implementation checklist

---

## Instructions for Claude

When this command is invoked with a URL:

### Step 1: Fetch & Analyze

Use bash (curl) to fetch the URL and extract:

**Visual Design:**
- Overall aesthetic (modern SaaS, minimal, playful, corporate, etc.)
- Design philosophy (clean, dense, playful, professional)
- Distinctive visual characteristics

**Color Palette:**
- Primary color (hex code)
- Accent color (hex code)
- Background color (hex code)
- Text colors (hex codes)
- Any gradients or color combinations

**Typography:**
- Font families (headlines, body, code/monospace if present)
- Font sizes (in px or rem)
- Font weights used
- Line heights
- Text hierarchy approach

**Layout System:**
- Grid structure (columns, rows)
- Spacing philosophy (generous, tight, balanced)
- Section organization (hero, features, pricing, etc.)
- Responsive patterns
- Max-width containers

**Components:**
- Navigation style (sticky, transparent, solid, etc.)
- Button designs (size, style, variants)
- Card designs (shadows, borders, hover effects)
- Form elements (inputs, selects, textareas)
- Special UI elements (badges, tags, scores, metrics)

**Unique Features:**
- What makes this design stand out?
- Signature visual elements
- Innovative patterns
- Distinctive interactions

### Step 2: Map to Known References

Compare the analyzed design to famous design systems AI models know well:

| Reference | When to Use |
|-----------|-------------|
| "Linear-inspired" | Minimal, smooth, dark theme, purple/violet accents |
| "Vercel-style" | Clean, developer-focused, modern, subtle |
| "Stripe-inspired" | Professional, trustworthy, polished, blue accents |
| "Notion-like" | Functional, clean, content-focused |
| "ProductHunt-style" | Card grid, metrics, voting, orange accents |
| "Framer-inspired" | Bold typography, animations, creative |
| "Supabase-style" | Vibrant gradients, green accents, modern |
| "Railway-inspired" | Dark mode, neon accents, technical |
| "Cal.com-style" | Open-source aesthetic, clean, blue/purple |

Identify which combination best matches the analyzed design.

### Step 3: Generate Prompt Templates

Create three versions of prompts:

#### **A. One-Liner (Quick Trigger):**
```
"[Famous reference]-inspired [aesthetic] with [unique elements]"
```
Example: "ProductHunt-inspired card grid with scoring badges and blue accents"

#### **B. Medium Prompt (Balanced):**
```
"Build a [famous reference]-inspired [aesthetic] interface featuring:
- [Key layout pattern]
- [Distinctive components]
- [Color scheme with hex codes]
- [Typography approach]
Use [design system] with [framework]."
```

#### **C. Full Prompt (Comprehensive):**
```
"Build a [description] using [famous reference]-inspired design with:

VISUAL STYLE:
- Overall aesthetic: [description]
- Design philosophy: [approach]

COLOR PALETTE:
- Primary: [hex]
- Accent: [hex]
- Background: [hex]
- Text: [hex]

TYPOGRAPHY:
- Headlines: [font], [size]px, [weight]
- Body: [font], [size]px, line-height [ratio]
- Code/Special: [font if applicable]

LAYOUT:
- [Grid/flex system]
- [Spacing approach]
- [Section structure]
- [Responsive behavior]

COMPONENTS:
- Navigation: [style]
- Buttons: [design]
- Cards: [structure]
- Forms: [approach]
- [Any unique elements]

UNIQUE FEATURES:
- [Distinctive element 1]
- [Distinctive element 2]
- [Distinctive element 3]

Use [design system like shadcn/ui] with [framework like Tailwind CSS].
Reference: [Famous site 1]'s [aspect], [Famous site 2]'s [aspect]."
```

### Step 4: Provide Improvement Suggestions

Analyze what could be enhanced:

**UX Improvements:**
- [Suggestion for better user experience]

**Visual Enhancements:**
- [Suggestion for modern aesthetics]

**Accessibility:**
- [Suggestion for better accessibility]

**Performance:**
- [Suggestion for faster loading]

**Additional Features:**
- [Ideas for valuable additions]

### Step 5: Create Implementation Checklist

Provide actionable steps:

```markdown
## 🚀 Implementation Checklist

### Phase 1: Design System Setup
- [ ] Set up color tokens (primary, accent, background)
- [ ] Configure typography (fonts, sizes, weights)
- [ ] Define spacing scale (margins, padding)

### Phase 2: Core Layout
- [ ] Build navigation component
- [ ] Create hero section
- [ ] Implement grid/card system

### Phase 3: Components
- [ ] Design button variants
- [ ] Build card components
- [ ] Create form elements
- [ ] [Any unique components]

### Phase 4: Polish
- [ ] Add hover/focus states
- [ ] Implement responsive design
- [ ] Test accessibility
- [ ] Optimize performance
```

---

## Output Format

Present findings using this structure:

```markdown
# 🎨 Design Analysis: [Website Name]

## 1️⃣ Design Summary

**Visual Style:** [Description]
**Best Described As:** [Famous reference] + [Famous reference]

### Color Palette
- **Primary:** [hex]
- **Accent:** [hex]
- **Background:** [hex]
- **Text:** [hex]

### Typography
- **Headlines:** [font], [size]px, [weight]
- **Body:** [font], [size]px, line-height [ratio]

### Layout Pattern
[Description of grid, sections, spacing]

### Unique Features
- [Feature 1]
- [Feature 2]
- [Feature 3]

---

## 2️⃣ AI-Friendly Prompts

### ⚡ One-Liner (Quick Trigger)
```
[Short prompt]
```

### 📋 Medium Prompt
```
[Balanced prompt]
```

### 📝 Full Prompt (Copy-Paste Ready)
```
[Comprehensive prompt]
```

---

## 3️⃣ Improvement Opportunities

### What's Already Great
- [Positive aspect 1]
- [Positive aspect 2]

### What Could Be Better
- [Improvement 1]
- [Improvement 2]

### Modern Enhancements
- [Enhancement 1]
- [Enhancement 2]

---

## 4️⃣ Implementation Checklist

[Actionable checklist as defined above]

---

## 5️⃣ Quick Reference Card

**Copy this for future use:**

| Element | Specification |
|---------|---------------|
| **Style** | [Reference-inspired aesthetic] |
| **Colors** | [Primary hex], [Accent hex] |
| **Fonts** | [Font families] |
| **Layout** | [Pattern description] |
| **Components** | [Key elements] |
| **Framework** | [Suggested framework] |

**One-line trigger:** `[Short prompt here]`

---

**Analysis complete! Use the prompts above with any AI builder (Anything, v0, etc.)**
```

---

## Examples

### Example 1: Analyzing Linear
```bash
/design:analyze https://linear.app
```

**Expected Output:**
- Style: Minimal, smooth, professional
- Colors: Dark navy background, purple accents
- Layout: Clean sections, generous whitespace
- Prompt: "Linear-inspired modern SaaS with dark theme..."

### Example 2: Analyzing Ideabrowser
```bash
/design:analyze https://ideabrowser.com
```

**Expected Output:**
- Style: Professional data-focused
- Colors: White background, blue (#5179F0) accents
- Layout: Card grid with metrics
- Prompt: "ProductHunt-inspired card layout with scoring badges..."

### Example 3: Analyzing Competitor
```bash
/design:analyze https://competitor-product.com
```

**Expected Output:**
- Complete design breakdown
- AI prompts to build similar UI
- Suggestions to make yours better

---

## Safety Guarantees

This command is **100% safe** because:

✅ **Read-only:** Only fetches external websites (no file modifications)
✅ **No Git operations:** Doesn't touch repository state
✅ **No Bash commands:** Pure analysis and output
✅ **No writes:** Doesn't create/edit files in repo
✅ **Stateless:** Works regardless of current directory or repo state
✅ **User-scoped:** Available everywhere, affects nothing

**You can run this anytime, anywhere, in any repo condition.**

---

## Use Cases

### 🎯 Building New Products
```bash
# Find inspiration from the best
/design:analyze https://linear.app
/design:analyze https://stripe.com
# Get AI prompts to build similar quality
```

### 🔍 Competitive Analysis
```bash
# Study competitor designs
/design:analyze https://competitor.com
# Extract their patterns
# Build yours better
```

### 📚 Learning Design
```bash
# Analyze award-winning sites
/design:analyze https://awwwards-winner.com
# Understand what makes great design
# Build pattern library
```

### ⚡ Quick Prototyping
```bash
# See a design you like
/design:analyze https://cool-site.com
# Get instant AI prompts
# Paste into builder (Anything, v0, etc.)
```

---

## Tips for Best Results

1. **Analyze multiple references:** Run command on 2-3 similar sites, compare patterns
2. **Focus on patterns, not pixels:** Use generated prompts as starting points
3. **Combine suggestions:** Mix elements from different analyses
4. **Add your twist:** Use improvements section to make it unique
5. **Test prompts:** Try generated prompts in different AI builders

---

## Notes

- Works with any publicly accessible website
- Some sites may block automated fetching (rare)
- Always respect original designs (inspiration, not theft)
- Generated prompts are starting points, not exact replicas
- Consider accessibility and performance in your adaptations

---

**Built with Ihsan** | User-scoped command | Safe to run anywhere | No side effects

---

**May this tool help you build beautiful interfaces that serve users well. Aamiin.**
