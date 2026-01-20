# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a static HTML form for Least of These Carolinas (LOTC), a non-profit organization that supports children in foster and kinship care. The form collects intake requests for various programs including Bags of Hope, Birthday, General requests, Just Like You, and Life Box.

## Architecture

**Single-Page Static Form**
- Self-contained HTML file (`lotc-request-form.html`) with embedded CSS and JavaScript
- No build process, dependencies, or server-side logic
- Form validation and submission handling occurs entirely in the browser
- Currently logs form data to console and shows an alert (production implementation would require backend integration)

**Brand Assets**
- `LOTC Brand Guideline.pdf` - Official brand guidelines including colors, fonts, and logo usage
- `Request Form - Apricot.pdf` - Reference document showing the original form requirements

## Brand Colors (from guidelines)

- **Primary Red**: `#c22035` - Used for headings, buttons, and brand accents
- **Light Blue**: `#86b2d3` - Used for background and borders
- **Dark**: `#060511` - Used for body text
- **Light Gray**: `#a7a8a3` - Used for secondary text
- **White**: `#ffffff` - Used for form container background

## Form Structure

The form collects information across six main sections:

1. **Form Completed By** - Name of person submitting the form
2. **Caregiver Details** - Contact information and relationship to child
3. **Social Worker Details** - DSS social worker contact information
4. **Child Details** - Demographics, placement type, request type, and pickup preferences
5. **Child's Clothing Sizes** - Shirt, pants, shoes, undergarments, and diaper sizes
6. **Additional Information** - Favorite colors, interests, and specific needs

## Development

Since this is a static HTML file with no build process:

- **View the form**: Open `lotc-request-form.html` directly in a browser
- **Edit the form**: Modify the HTML file directly - changes are immediately visible on reload
- **No compilation or build steps required**

## Key Implementation Notes

**Form Submission**
- Currently prevents default submission and logs data to console (lines 482-497)
- Production implementation will need backend endpoint to process submissions
- Form includes privacy policy agreement checkbox (required)

**Client-Side Features**
- Age calculation from date of birth (lines 500-511)
- Form validation using HTML5 required attributes
- Responsive design with mobile breakpoint at 768px

**Typography**
- Uses Google Fonts: Poppins (weights 400, 700)
- Font choices align with LOTC brand guidelines

## Integration Points for Future Development

When connecting this form to a backend system:

1. Replace the `console.log` in the form submit handler (line 490) with an API call
2. Add error handling for failed submissions
3. Consider adding loading states during submission
4. Implement server-side validation to match client-side rules
5. Add confirmation email functionality
6. Store submitted data in a database or CRM system (e.g., Apricot)
