# LOTC Request Form - Implementation Specification

## Executive Summary

This specification outlines improvements to the LOTC intake form to align with brand guidelines, implement intelligent auto-fill functionality, and prepare for Neon CRM integration.

**Current Status:** Static HTML form with basic validation
**Target State:** Dynamic, intelligent form with auto-calculations and CRM-ready data structure

---

## 1. UI/UX Improvements

### 1.1 Typography Corrections

**Issue:** Missing Oakside font for main headline per brand guidelines

**Fix Required:**
```html
<!-- Add to <head> -->
<link href="https://fonts.googleapis.com/css2?family=Poppins:wght@400;700&display=swap" rel="stylesheet">
<!-- Add Oakside font (need to source/host this custom font) -->
```

**Typography Hierarchy:**
- **Main H1 "Request Form"**: Oakside font (brand guideline requirement)
- **Section titles (.section-title)**: Poppins Bold ✓ (already correct)
- **Labels**: Poppins Bold ✓ (already correct)
- **Body text/inputs**: Poppins Regular ✓ (already correct)
- **Buttons**: Poppins Bold ✓ (already correct)

**Action Items:**
1. Source/license Oakside font or use closest Google Fonts alternative (e.g., "Bebas Neue" or "Archivo Black")
2. Update h1 CSS to use Oakside
3. Document font fallback strategy

### 1.2 Visual Enhancements

**Current Issues:**
- Form feels dense - could use more breathing room
- No visual feedback during interactions
- Missing progress indicators for long form

**Recommended Improvements:**
1. **Section Numbering**: Add visual section indicators (1/6, 2/6, etc.)
2. **Micro-interactions**:
   - Input focus animations (subtle scale/shadow)
   - Success checkmarks when required fields are filled
   - Smooth transitions between states
3. **Responsive Logo**: SVG logo should be external file for easier updates
4. **Loading States**: Add skeleton screens for when calculating values

**Implementation Priority:** Medium (Phase 2)

---

## 2. Dynamic Field Logic

### 2.1 Auto-Calculate Child's Age

**Current State:** Age calculation exists but only logs to console (line 500-511)

**Enhancement Required:**
```javascript
// Add visible age display field
<div class="form-group">
    <label>Child's Age (Auto-calculated)</label>
    <input type="text" id="childAge" name="childAge" readonly
           style="background-color: #f5f5f5; cursor: not-allowed;">
</div>

// Enhanced calculation with display update
document.getElementById('childDOB').addEventListener('change', function() {
    const age = calculateAge(this.value);
    document.getElementById('childAge').value = age + ' years old';

    // Trigger estimated value calculation
    calculateEstimatedValue();
});
```

**Where to Place:** After "Child's Date of Birth" field in Child Details section

### 2.2 Auto-Calculate Estimated Value

**Pricing Structure (from IMG_4970):**

#### Bags of Hope (BOH) Values by Age:
| Age Range | Value |
|-----------|-------|
| 1-2 (Baby) | $250 |
| 3-5 (Toddler) | $250 |
| 6-11 (Child) | $250 |
| 12+ (Teen) | $300 |

#### Birthday Value:
| Fixed Value |
|-------------|
| $30 |

#### Coats Values by Size:
| Size | Value |
|------|-------|
| Baby sizes | $25 |
| Toddler sizes | $30 |
| Child sizes | $35-40 |
| Adult sizes | $40 |

#### Shoes Values by Size:
| Size | Value |
|------|-------|
| Baby sizes | $30 |
| Toddler sizes | $30 |
| Child sizes | $50 |
| Adult sizes | $65 |

**Implementation Logic:**

```javascript
// Pricing configuration object
const PRICING_CONFIG = {
    'bags-of-hope': {
        ageRanges: [
            { min: 1, max: 2, value: 250, label: 'Baby' },
            { min: 3, max: 5, value: 250, label: 'Toddler' },
            { min: 6, max: 11, value: 250, label: 'Child' },
            { min: 12, max: 18, value: 300, label: 'Teen' }
        ]
    },
    'birthday': {
        flatValue: 30
    },
    'general': {
        // TBD - may need user input or selection
        requiresManualEntry: true
    },
    'just-like-you': {
        // TBD - scholarship amount varies
        requiresManualEntry: true
    },
    'life-box': {
        // TBD - may be age-based like BOH
        requiresManualEntry: true
    }
};

function calculateEstimatedValue() {
    const requestType = document.getElementById('requestType').value;
    const dob = document.getElementById('childDOB').value;

    if (!requestType || !dob) return;

    const age = calculateAge(dob);
    const estimatedValueField = document.getElementById('estimatedValue');

    if (requestType === 'bags-of-hope') {
        const ageRange = PRICING_CONFIG['bags-of-hope'].ageRanges.find(
            range => age >= range.min && age <= range.max
        );

        if (ageRange) {
            estimatedValueField.value = `$${ageRange.value}`;
            estimatedValueField.dataset.category = ageRange.label;
        }
    } else if (requestType === 'birthday') {
        estimatedValueField.value = '$30';
    } else {
        // For other types, enable manual entry
        estimatedValueField.value = '';
        estimatedValueField.placeholder = 'Enter estimated value';
        estimatedValueField.readOnly = false;
    }
}

// Trigger calculation when either field changes
document.getElementById('requestType').addEventListener('change', calculateEstimatedValue);
document.getElementById('childDOB').addEventListener('change', calculateEstimatedValue);
```

**New Field to Add:**
```html
<!-- Add to Child Details section, after Request Type -->
<div class="form-group">
    <label for="estimatedValue">Estimated Value (Auto-calculated)</label>
    <input type="text" id="estimatedValue" name="estimatedValue" readonly>
    <div class="note" id="valueNote" style="display: none;">
        Based on <span id="valueCategory"></span> category
    </div>
</div>
```

### 2.3 Smart Age Category Detection

**Purpose:** Display which age category the child falls into for transparency

**Implementation:**
```javascript
function displayAgeCategory(age) {
    let category = '';

    if (age >= 1 && age <= 2) category = 'Baby (1-2 years)';
    else if (age >= 3 && age <= 5) category = 'Toddler (3-5 years)';
    else if (age >= 6 && age <= 11) category = 'Child (6-11 years)';
    else if (age >= 12) category = 'Teen (12+ years)';
    else category = 'Infant (under 1 year)';

    const categoryDisplay = document.getElementById('ageCategoryDisplay');
    if (categoryDisplay) {
        categoryDisplay.textContent = category;
        categoryDisplay.style.color = '#c22035';
        categoryDisplay.style.fontWeight = '600';
    }
}
```

---

## 3. Smart Field Visibility

### 3.1 Request Type Conditional Fields

**Logic:** Different request types require different information

#### Bags of Hope Specific Fields:
- Estimated Value (auto-calculated)
- Clothing sizes (all fields)
- "Reason a bed is needed" (if applicable)

#### Birthday Specific Fields:
- Estimated Value ($30 flat)
- Favorite Color
- Interests
- Simplified needs list

#### General Request Fields:
- Manual estimated value entry
- Detailed needs description
- May not need all clothing sizes

#### Just Like You Scholarship Fields:
- Activity/program details
- Scholarship amount (manual entry)
- Timeline/schedule information

#### Life Box Fields:
- Similar to Bags of Hope
- May include transition support items

**Implementation:**

```javascript
const FIELD_VISIBILITY_RULES = {
    'bags-of-hope': {
        show: ['clothingSizes', 'estimatedValue', 'needs', 'interests'],
        hide: ['activityDetails'],
        require: ['childGender', 'childDOB', 'clothingSizes']
    },
    'birthday': {
        show: ['estimatedValue', 'favoriteColor', 'interests'],
        hide: ['clothingSizes', 'activityDetails'],
        require: ['childDOB', 'favoriteColor']
    },
    'general': {
        show: ['needs', 'estimatedValue'],
        hide: ['activityDetails'],
        require: ['needs']
    },
    'just-like-you': {
        show: ['activityDetails', 'estimatedValue'],
        hide: ['clothingSizes', 'diaperSize'],
        require: ['activityDetails']
    },
    'life-box': {
        show: ['clothingSizes', 'estimatedValue', 'needs'],
        hide: ['activityDetails'],
        require: ['childGender', 'childDOB']
    }
};

function updateFieldVisibility(requestType) {
    const rules = FIELD_VISIBILITY_RULES[requestType];
    if (!rules) return;

    // Hide all conditional sections first
    document.querySelectorAll('.conditional-section').forEach(section => {
        section.style.display = 'none';
    });

    // Show relevant sections
    rules.show.forEach(sectionId => {
        const section = document.getElementById(sectionId + 'Section');
        if (section) {
            section.style.display = 'block';
        }
    });

    // Update required attributes
    rules.require.forEach(fieldId => {
        const field = document.getElementById(fieldId);
        if (field) field.required = true;
    });
}

// Trigger on request type change
document.getElementById('requestType').addEventListener('change', function() {
    updateFieldVisibility(this.value);
    calculateEstimatedValue();
});
```

**HTML Structure Update Required:**

Wrap conditional sections:
```html
<div id="clothingSizesSection" class="section conditional-section">
    <h2 class="section-title">Child's Clothing Sizes</h2>
    <!-- existing content -->
</div>
```

---

## 4. Missing Fields from Apricot Form

### 4.1 Critical Missing Fields

**From Apricot comparison, these fields should be added:**

#### Form Completed By Section Enhancement:
```html
<!-- After completedBy name field -->
<div class="form-group">
    <label for="completedByRole">Your Role/Relationship</label>
    <select id="completedByRole" name="completedByRole">
        <option value="">Select your role...</option>
        <option value="caregiver">I am the Caregiver</option>
        <option value="social-worker">I am the Social Worker</option>
        <option value="other">Other (Family Member, Advocate, etc.)</option>
    </select>
</div>
```

#### Request Details Section (Internal/Administrative - Hidden by Default):
```html
<div class="section" id="requestDetailsSection" style="display: none;">
    <h2 class="section-title">Request Details (For Office Use)</h2>

    <div class="form-group">
        <label for="requestDate">Request Date</label>
        <input type="date" id="requestDate" name="requestDate" readonly>
    </div>

    <div class="form-group">
        <label for="requestStatus">Request Status</label>
        <select id="requestStatus" name="requestStatus">
            <option value="new">New</option>
            <option value="in-progress">In Progress</option>
            <option value="packed">Packed</option>
            <option value="ready">Ready for Pickup</option>
            <option value="delivered">Delivered</option>
            <option value="closed">Closed</option>
        </select>
    </div>

    <div class="form-group">
        <label for="estimatedValue">Estimated Value</label>
        <input type="text" id="estimatedValue" name="estimatedValue" readonly>
    </div>

    <div class="form-group">
        <label for="internalNotes">Internal Notes</label>
        <textarea id="internalNotes" name="internalNotes"></textarea>
    </div>
</div>
```

#### County Consistency:
Both Caregiver and Social Worker have county fields - ensure these are used for geographic reporting to Neon CRM.

### 4.2 Optional Enhancement: Record Linking

**Future Feature:** Link to existing caregiver/social worker/child records

This would require:
1. Local storage or API lookup of existing records
2. Auto-fill functionality when selecting existing contact
3. "Use Existing" vs "Add New" toggle buttons

**Implementation Priority:** Phase 3 (after backend integration)

---

## 5. Neon CRM Integration Strategy

### 5.1 Data Mapping

**Neon CRM Field Mapping Structure:**

```javascript
const NEON_CRM_MAPPING = {
    // Caregiver maps to -> Contact/Account
    caregiver: {
        firstName: 'caregiverName', // Will need to split
        lastName: 'caregiverName',  // Will need to split
        phone: 'caregiverPhone',
        email: 'caregiverEmail',
        county: 'caregiverCounty',
        canText: 'caregiverText',
        isLicensed: 'licensed',
        relationship: 'relationshipToChild'
    },

    // Social Worker maps to -> Contact
    socialWorker: {
        firstName: 'socialWorkerName', // Will need to split
        lastName: 'socialWorkerName',
        phone: 'socialWorkerPhone',
        email: 'socialWorkerEmail',
        county: 'socialWorkerCounty',
        canText: 'socialWorkerText'
    },

    // Child maps to -> Participant/Constituent
    child: {
        firstName: 'childFirstName',
        lastInitial: 'childLastInitial',
        nickname: 'childNickname',
        gender: 'childGender',
        dateOfBirth: 'childDOB',
        age: 'calculated',
        ethnicity: 'childEthnicity',
        placementType: 'placementType',
        custodyCounty: 'custodyCounty',
        siblings: 'siblingsNames',
        siblingsInHome: 'siblingsSameHome'
    },

    // Request maps to -> Donation/Program Enrollment
    request: {
        requestType: 'requestType',
        completionContact: 'completionContact',
        pickupLocation: 'pickupLocation',
        requestDate: 'auto-generated',
        status: 'auto-set-to-new',
        estimatedValue: 'calculated',

        // Custom fields
        favoriteColor: 'favoriteColor',
        interests: 'interests',
        needs: 'needs',

        // Clothing sizes
        shirtSize: 'shirtSize',
        pantSize: 'pantSize',
        shoeSize: 'shoeSize',
        undergarmentSize: 'undergarmentSize',
        diaperSize: 'diaperSize'
    },

    // Consent/Privacy
    privacy: {
        agreedToPrivacyPolicy: 'privacyAgreement',
        agreedDate: 'auto-generated'
    }
};
```

### 5.2 Data Validation Before Submission

**Pre-submission Checklist:**

```javascript
function validateForNeonCRM(formData) {
    const validationErrors = [];

    // 1. Required fields check
    if (!formData.caregiverName || !formData.caregiverEmail) {
        validationErrors.push('Caregiver contact information is incomplete');
    }

    if (!formData.socialWorkerName || !formData.socialWorkerEmail) {
        validationErrors.push('Social Worker contact information is incomplete');
    }

    if (!formData.childFirstName || !formData.childDOB) {
        validationErrors.push('Child information is incomplete');
    }

    // 2. Email format validation
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(formData.caregiverEmail)) {
        validationErrors.push('Caregiver email format is invalid');
    }
    if (!emailRegex.test(formData.socialWorkerEmail)) {
        validationErrors.push('Social Worker email format is invalid');
    }

    // 3. Phone format validation (basic)
    const phoneRegex = /^\d{3}-\d{3}-\d{4}$/;
    if (!phoneRegex.test(formData.caregiverPhone)) {
        validationErrors.push('Caregiver phone should be in format: 123-456-7890');
    }

    // 4. Date validation
    const dob = new Date(formData.childDOB);
    const today = new Date();
    if (dob >= today) {
        validationErrors.push('Child date of birth cannot be in the future');
    }

    // 5. Age-appropriate validation
    const age = calculateAge(formData.childDOB);
    if (age > 18 && formData.requestType === 'bags-of-hope') {
        validationErrors.push('Bags of Hope is for children under 18');
    }

    return {
        isValid: validationErrors.length === 0,
        errors: validationErrors
    };
}
```

### 5.3 Submission Payload Structure

**Recommended JSON structure for API submission:**

```javascript
function buildNeonCRMPayload(formData) {
    return {
        apiVersion: '2.1', // Adjust based on your Neon CRM API version
        timestamp: new Date().toISOString(),

        request: {
            requestId: generateRequestId(), // Auto-generate unique ID
            requestType: formData.requestType,
            requestDate: new Date().toISOString().split('T')[0],
            status: 'new',
            estimatedValue: parseFloat(formData.estimatedValue.replace('$', '')),
            completionContact: formData.completionContact,
            pickupLocation: formData.pickupLocation,
            privacyConsent: formData.privacyAgreement === 'on'
        },

        caregiver: {
            name: formData.caregiverName,
            phone: formData.caregiverPhone,
            email: formData.caregiverEmail,
            county: formData.caregiverCounty,
            canReceiveTexts: formData.caregiverText === 'yes',
            isLicensedFosterParent: formData.licensed === 'yes',
            relationshipToChild: formData.relationshipToChild
        },

        socialWorker: {
            name: formData.socialWorkerName,
            phone: formData.socialWorkerPhone,
            email: formData.socialWorkerEmail,
            county: formData.socialWorkerCounty,
            canReceiveTexts: formData.socialWorkerText === 'yes'
        },

        child: {
            firstName: formData.childFirstName,
            lastInitial: formData.childLastInitial,
            nickname: formData.childNickname,
            gender: formData.childGender,
            dateOfBirth: formData.childDOB,
            age: calculateAge(formData.childDOB),
            ethnicity: formData.childEthnicity,
            placementType: formData.placementType,
            custodyCounty: formData.custodyCounty,
            siblings: {
                names: formData.siblingsNames,
                inSameHome: formData.siblingsSameHome === 'yes'
            }
        },

        preferences: {
            favoriteColor: formData.favoriteColor,
            interests: formData.interests,
            needs: formData.needs
        },

        clothingSizes: {
            shirt: formData.shirtSize,
            pants: formData.pantSize,
            shoes: formData.shoeSize,
            undergarments: formData.undergarmentSize,
            diapers: formData.diaperSize
        },

        metadata: {
            submittedBy: formData.completedBy,
            submissionTimestamp: new Date().toISOString(),
            userAgent: navigator.userAgent,
            formVersion: '1.0'
        }
    };
}
```

### 5.4 Interim Solution: Local Storage

**Until backend is ready, save submissions to localStorage:**

```javascript
function saveToLocalStorage(payload) {
    const submissions = JSON.parse(localStorage.getItem('lotcSubmissions') || '[]');

    submissions.push({
        ...payload,
        localId: `local-${Date.now()}`,
        syncStatus: 'pending',
        savedAt: new Date().toISOString()
    });

    localStorage.setItem('lotcSubmissions', JSON.stringify(submissions));

    // Also save as CSV-ready format for manual export
    saveAsCSV(payload);
}

function saveAsCSV(payload) {
    // Flatten the nested object for CSV export
    const flatData = {
        'Request ID': payload.request.requestId,
        'Request Type': payload.request.requestType,
        'Request Date': payload.request.requestDate,
        'Estimated Value': payload.request.estimatedValue,
        'Child First Name': payload.child.firstName,
        'Child Last Initial': payload.child.lastInitial,
        'Child DOB': payload.child.dateOfBirth,
        'Child Age': payload.child.age,
        'Child Gender': payload.child.gender,
        'Caregiver Name': payload.caregiver.name,
        'Caregiver Email': payload.caregiver.email,
        'Caregiver Phone': payload.caregiver.phone,
        'Social Worker Name': payload.socialWorker.name,
        'Social Worker Email': payload.socialWorker.email,
        // ... add all other fields
    };

    const csv = Object.entries(flatData)
        .map(([key, value]) => `"${key}","${value}"`)
        .join('\n');

    localStorage.setItem('lotcSubmissionsCSV',
        localStorage.getItem('lotcSubmissionsCSV') + '\n' + csv
    );
}
```

### 5.5 Export Utility

**Add admin panel to export submissions:**

```html
<!-- Add hidden admin panel (toggle with keyboard shortcut) -->
<div id="adminPanel" style="display: none;" class="admin-panel">
    <h2>Pending Submissions</h2>
    <button onclick="exportToCSV()">Export All to CSV</button>
    <button onclick="clearSubmissions()">Clear All</button>
    <div id="submissionList"></div>
</div>

<script>
// Press Ctrl+Alt+A to toggle admin panel
document.addEventListener('keydown', function(e) {
    if (e.ctrlKey && e.altKey && e.key === 'a') {
        const panel = document.getElementById('adminPanel');
        panel.style.display = panel.style.display === 'none' ? 'block' : 'none';
        loadSubmissionList();
    }
});

function exportToCSV() {
    const csv = localStorage.getItem('lotcSubmissionsCSV');
    const blob = new Blob([csv], { type: 'text/csv' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `lotc-submissions-${new Date().toISOString().split('T')[0]}.csv`;
    a.click();
}
</script>
```

---

## 6. Implementation Phases

### Phase 1: Core Dynamic Features (Week 1-2)
**Priority: HIGH**

1. ✅ Auto-calculate child age from DOB
2. ✅ Display age visually in form
3. ✅ Auto-calculate estimated value based on request type and age
4. ✅ Implement smart field visibility based on request type
5. ✅ Add pricing configuration object
6. ✅ Add estimated value field with read-only display

**Deliverables:**
- Updated `lotc-request-form.html` with dynamic features
- JavaScript module for calculations
- Basic validation enhancements

**Testing Checklist:**
- [ ] Age calculates correctly for all DOB inputs
- [ ] Estimated value populates correctly for BOH (all age ranges)
- [ ] Estimated value shows $30 for Birthday requests
- [ ] Fields show/hide correctly based on request type
- [ ] Form still works without JavaScript (graceful degradation)

### Phase 2: UI/UX Enhancements (Week 3)
**Priority: MEDIUM**

1. ✅ Add Oakside font for main headline
2. ✅ Implement micro-interactions (focus states, transitions)
3. ✅ Add section progress indicators
4. ✅ Improve mobile responsive behavior
5. ✅ Add loading states for calculations
6. ✅ Implement success/error message system

**Deliverables:**
- Enhanced CSS with animations
- Progress indicator component
- Mobile-optimized layout

### Phase 3: Missing Fields & Data Completeness (Week 4)
**Priority: MEDIUM**

1. ✅ Add "Form Completed By" role field
2. ✅ Add estimated value to Request Details section
3. ✅ Implement proper name splitting (first/last) for CRM
4. ✅ Add internal notes field
5. ✅ Add request status tracking
6. ✅ Implement phone number formatting

**Deliverables:**
- Complete field mapping document
- Updated form with all Apricot fields
- Validation rules for all fields

### Phase 4: Local Storage & Export (Week 5)
**Priority: HIGH (for interim solution)**

1. ✅ Implement localStorage submission queue
2. ✅ Build CSV export functionality
3. ✅ Create admin panel for managing submissions
4. ✅ Add submission counter/status indicator
5. ✅ Implement "draft save" feature

**Deliverables:**
- Working localStorage system
- CSV export utility
- Admin panel for LOTC staff

### Phase 5: Neon CRM Integration (Week 6+)
**Priority: HIGH (requires backend)**

1. ⏳ Build backend API endpoint
2. ⏳ Implement Neon CRM API authentication
3. ⏳ Create data transformation layer
4. ⏳ Build sync mechanism for localStorage queue
5. ⏳ Add error handling & retry logic
6. ⏳ Implement success/failure notifications

**Deliverables:**
- Backend service (Node.js recommended)
- Neon CRM integration module
- Sync dashboard
- Error logging system

---

## 7. Technical Specifications

### 7.1 Browser Support

**Target Browsers:**
- Chrome/Edge: Latest 2 versions
- Firefox: Latest 2 versions
- Safari: Latest 2 versions
- Mobile Safari (iOS): 14+
- Chrome Mobile (Android): Latest 2 versions

**Graceful Degradation:**
- Core form submission must work without JavaScript
- CSS Grid with Flexbox fallback
- HTML5 validation as primary, JS as enhancement

### 7.2 Performance Targets

- Initial page load: < 2 seconds
- Time to Interactive: < 3 seconds
- Form validation: < 100ms response time
- Auto-calculate functions: < 50ms
- Lighthouse score: > 90

### 7.3 Accessibility Requirements

**WCAG 2.1 Level AA Compliance:**
- All form fields must have associated labels
- Color contrast ratio: minimum 4.5:1
- Keyboard navigation: full form completable without mouse
- Screen reader friendly: proper ARIA labels
- Focus indicators: visible and high contrast
- Error messages: clear and associated with fields

**Implementation:**
```html
<!-- Example accessible field -->
<div class="form-group" role="group" aria-labelledby="cgNameLabel">
    <label id="cgNameLabel" for="caregiverName">
        Caregiver's Name
        <span aria-label="required" class="required">*</span>
    </label>
    <input
        type="text"
        id="caregiverName"
        name="caregiverName"
        required
        aria-required="true"
        aria-describedby="cgNameHelp"
        aria-invalid="false"
    >
    <small id="cgNameHelp" class="help-text">
        Enter the full legal name of the caregiver
    </small>
    <div id="cgNameError" class="error-message" role="alert" aria-live="polite"></div>
</div>
```

### 7.4 Security Considerations

**Data Protection:**
1. **PII Handling**: Child's name limited to first name + last initial only
2. **Input Sanitization**: All user inputs must be sanitized before storage
3. **XSS Prevention**: Use textContent instead of innerHTML where possible
4. **HTTPS Only**: Form must be served over HTTPS in production
5. **Privacy Policy**: Required acknowledgment before submission

**Validation:**
```javascript
function sanitizeInput(input) {
    const temp = document.createElement('div');
    temp.textContent = input;
    return temp.innerHTML;
}

function validateInput(field, value) {
    // Remove any HTML tags
    value = sanitizeInput(value);

    // Field-specific validation
    switch(field) {
        case 'email':
            return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value);
        case 'phone':
            return /^\d{3}-\d{3}-\d{4}$/.test(value);
        case 'name':
            return /^[a-zA-Z\s'-]{2,50}$/.test(value);
        default:
            return true;
    }
}
```

---

## 8. Testing Strategy

### 8.1 Unit Tests

**Functions to Test:**
- `calculateAge(dob)` - returns correct age for various dates
- `calculateEstimatedValue(requestType, age)` - returns correct values
- `updateFieldVisibility(requestType)` - shows/hides correct fields
- `validateInput(field, value)` - validates correctly
- `buildNeonCRMPayload(formData)` - creates correct structure

**Testing Framework:** Jest or Mocha (for future backend)

### 8.2 Integration Tests

**Scenarios to Test:**
1. Complete form flow for each request type
2. Age calculation → Estimated value auto-population
3. Request type change → Field visibility update
4. Form submission → localStorage save
5. localStorage → CSV export

### 8.3 User Acceptance Testing (UAT)

**Test Scenarios:**

| Scenario | Steps | Expected Result |
|----------|-------|-----------------|
| BOH Request for Baby | Enter DOB for 1-year-old, select BOH | Shows $250 value, all clothing fields visible |
| Birthday Request | Select Birthday type | Shows $30 value, hides clothing sizes |
| Age Edge Cases | Enter DOB for exactly 12-year-old | Shows $300 (Teen category) |
| Mobile Completion | Fill entire form on mobile device | All fields accessible, form submits successfully |
| Field Validation | Submit with missing required fields | Clear error messages, focus on first error |

### 8.4 Browser Testing Matrix

| Browser | Desktop | Mobile | Expected Issues |
|---------|---------|--------|-----------------|
| Chrome | ✅ | ✅ | None |
| Safari | ✅ | ✅ | Date picker styling may differ |
| Firefox | ✅ | ✅ | None |
| Edge | ✅ | N/A | None |
| Samsung Internet | N/A | ⚠️ | May need additional testing |

---

## 9. Documentation Requirements

### 9.1 User-Facing Documentation

**For Caregivers/Social Workers:**
1. **Form Completion Guide** (PDF)
   - Step-by-step screenshots
   - Common questions answered
   - What to do if technical issues arise
   - Privacy policy summary

2. **FAQ Document**
   - Q: What if I don't know all the information?
   - Q: Can I save and come back later?
   - Q: How long until I hear back?
   - Q: What if I need to update information?

### 9.2 Staff Documentation

**For LOTC Staff:**
1. **Admin Panel Guide**
   - How to access pending submissions
   - How to export to CSV
   - How to manually sync to Neon CRM (once available)
   - Troubleshooting common issues

2. **Data Dictionary**
   - Complete mapping of form fields to Neon CRM fields
   - Business logic for calculated fields
   - Valid values for dropdown fields

### 9.3 Developer Documentation

**For Future Developers:**
1. **Code Documentation** (JSDoc comments)
2. **Architecture Decision Records (ADRs)**
3. **API Integration Guide** (when backend is ready)
4. **Deployment Guide**

---

## 10. Future Enhancements

### 10.1 Short-term (3-6 months)
- Multi-language support (Spanish)
- Email notifications to caregiver/social worker
- QR code generation for tracking
- Photo upload capability (with privacy controls)
- Digital signature capture

### 10.2 Long-term (6-12 months)
- Mobile app version
- Caregiver portal (track request status)
- Inventory integration (check available items)
- Scheduling system for pickups
- Reporting dashboard for LOTC staff

---

## 11. Success Metrics

### 11.1 Technical Metrics
- Form completion rate: > 80%
- Form abandonment rate: < 20%
- Average completion time: < 10 minutes
- Mobile completion rate: > 40%
- Error rate: < 5%

### 11.2 Business Metrics
- Reduction in incomplete submissions: > 30%
- Staff time processing requests: -50%
- Data accuracy improvement: > 90%
- Neon CRM sync success rate: > 95%

### 11.3 User Satisfaction
- Post-submission survey: > 4/5 stars
- "Would recommend" rate: > 85%
- Support ticket volume: < 5 per week

---

## 12. Rollout Plan

### 12.1 Pilot Phase (2 weeks)
- Deploy to staging environment
- Test with 5 internal staff members
- Gather feedback
- Fix critical issues

### 12.2 Soft Launch (2 weeks)
- Deploy to production
- Notify select group of 10-15 trusted caregivers/SWs
- Monitor submissions closely
- Be ready to rollback if needed

### 12.3 Full Launch
- Announce via email, social media
- Update lotcarolinas.com with new form link
- Monitor metrics daily for first week
- Provide rapid support for issues

### 12.4 Post-Launch
- Weekly review of metrics for first month
- Monthly review thereafter
- Continuous improvement based on feedback
- Plan for Phase 5 (Neon CRM integration)

---

## Appendix A: File Structure

```
LOTCIntakeFORM/
├── lotc-request-form.html          # Main form file
├── js/
│   ├── form-validation.js          # Validation logic
│   ├── dynamic-fields.js           # Age & value calculations
│   ├── field-visibility.js         # Show/hide logic
│   ├── local-storage.js            # Submission queue
│   └── neon-crm-adapter.js         # CRM integration (future)
├── css/
│   ├── main.css                    # Current styles (extract from HTML)
│   ├── animations.css              # Micro-interactions
│   └── admin-panel.css             # Admin styles
├── fonts/
│   └── oakside/                    # Custom font files
├── docs/
│   ├── IMPLEMENTATION_SPEC.md      # This document
│   ├── USER_GUIDE.pdf              # For caregivers
│   ├── STAFF_GUIDE.pdf             # For LOTC staff
│   └── API_DOCS.md                 # Neon CRM integration
├── tests/
│   ├── unit/
│   └── integration/
└── CLAUDE.md                        # Repository guide
```

---

## Appendix B: Quick Reference - Request Type Rules

| Request Type | Estimated Value | Show Clothing Sizes | Show Activity Details | Show Favorite Color |
|--------------|-----------------|---------------------|----------------------|---------------------|
| Bags of Hope | Age-based ($250-300) | ✅ | ❌ | ⚠️ Optional |
| Birthday | $30 flat | ❌ | ❌ | ✅ |
| General | Manual entry | ⚠️ Conditional | ❌ | ⚠️ Optional |
| Just Like You | Manual entry | ❌ | ✅ | ⚠️ Optional |
| Life Box | Age-based (TBD) | ✅ | ❌ | ⚠️ Optional |

---

## Appendix C: Pricing Quick Reference

### Bags of Hope
- **1-2 years (Baby)**: $250
- **3-5 years (Toddler)**: $250
- **6-11 years (Child)**: $250
- **12+ years (Teen)**: $300

### Birthday
- **All ages**: $30

### Other Values (Reference)
**Coats**: $25-40 (by size)
**Shoes**: $30-65 (by size)

---

## Contact & Support

**Questions about this specification?**
- Technical: [Developer Email]
- Business Logic: Rebekah Estep (rebekahe@lotcarolinas.com)
- Implementation Timeline: [Project Manager]

**Document Version:** 1.0
**Last Updated:** January 20, 2026
**Next Review:** February 20, 2026
