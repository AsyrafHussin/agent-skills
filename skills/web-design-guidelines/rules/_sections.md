# Sections

This file defines all sections, their ordering, impact levels, and descriptions.
The section ID (in parentheses) is the filename prefix used to group rules.

---

## 1. Accessibility - Semantic Structure (a11y)

**Impact:** CRITICAL
**Description:** Semantic HTML and ARIA provide the foundation for accessible interfaces. Proper structure enables assistive technologies to understand and navigate content, directly impacting users with disabilities.

## 2. Accessibility - Keyboard & Focus (a11y)

**Impact:** CRITICAL
**Description:** Keyboard navigation and focus management are essential for users who cannot use a mouse. Missing or broken keyboard support excludes a significant portion of users.

## 3. Accessibility - Visual & Color (a11y)

**Impact:** CRITICAL
**Description:** Color contrast, text sizing, and visual indicators ensure content is perceivable by users with visual impairments or color blindness.

## 4. Forms - Input & Validation (form)

**Impact:** CRITICAL
**Description:** Proper form implementation with clear labels, autocomplete, and validation reduces errors and improves completion rates. Poor forms are the #1 cause of user frustration.

## 5. Forms - Error Handling (form)

**Impact:** HIGH
**Description:** Clear, accessible error messages help users recover from mistakes quickly. Poor error handling leads to form abandonment.

## 6. Forms - User Experience (form)

**Impact:** MEDIUM
**Description:** Advanced form patterns like multi-step flows and inline validation improve the user experience and increase conversion rates.

## 7. Animation & Motion (motion)

**Impact:** HIGH
**Description:** Respecting prefers-reduced-motion is critical for users with vestibular disorders. Improper animations can cause nausea, dizziness, or seizures.

## 8. Performance & UX (perf)

**Impact:** MEDIUM
**Description:** Performance directly impacts user experience. Images without dimensions cause layout shifts, lazy loading improves initial load time.
