# LOTC Request Form

A dynamic intake form for Least of These Carolinas to collect requests for their various programs including Bags of Hope, Birthday, General, Just Like You, and Life Box.

## 🚀 Quick Start

### Local Development
1. Simply open `lotc-request-form.html` in your browser
2. No build process or dependencies required

### Deploy to Vercel
```bash
# Login to Vercel (first time only)
vercel login

# Deploy to production
vercel --prod
```

## ✨ Features

### Dynamic Calculations
- **Auto-Calculate Age**: Automatically calculates child's age from date of birth
- **Auto-Calculate Value**: For Bags of Hope requests, automatically calculates estimated value based on age:
  - Ages 1-2 (Baby): $250
  - Ages 3-5 (Toddler): $250
  - Ages 6-11 (Child): $250
  - Ages 12+ (Teen): $300

### Smart Field Visibility
- **Bags of Hope**: Shows clothing sizes section + estimated value
- **Birthday**: Hides clothing sizes, no estimated value
- **Other Types**: Conditional sections based on request type

### Form Validation
- Real-time email validation
- Auto-format phone numbers (XXX-XXX-XXXX)
- Date validation (prevents future dates)
- Clear error messages

### Data Management
- **localStorage Queue**: All submissions saved locally until synced
- **CSV Export**: Press Ctrl+Alt+A (Command+Option+A on Mac) to access admin panel
- **Export Options**:
  - Export all submissions to CSV
  - View submissions in console
  - Clear all submissions

## 📁 Project Structure

```
LOTCIntakeFORM/
├── lotc-request-form.html          # Main form (self-contained)
├── vercel.json                     # Vercel deployment config
├── .vercelignore                   # Files to exclude from deployment
├── CLAUDE.md                       # Developer documentation
├── IMPLEMENTATION_SPEC.md          # Technical specifications
├── FEATURES_IMPLEMENTED.md         # Feature documentation
└── README.md                       # This file
```

## 🧪 Testing

### Test Bags of Hope Calculation
1. Select Request Type: "Bags of Hope"
2. Enter Child DOB: `2012-01-15` (makes them ~14 years old)
3. **Expected**: Age shows "14 years old - Teen", Value shows "$300"

### Test Field Visibility
1. Select Request Type: "Birthday"
2. **Expected**: Clothing sizes section disappears

### Test Admin Panel
1. Submit a form
2. Press Ctrl+Alt+A (Command+Option+A on Mac)
3. Choose option "1" to export CSV

## 🔐 Privacy & Security

- Child information limited to first name and last name
- All inputs sanitized before storage
- Privacy policy acknowledgment required
- No sensitive data in URLs or logs

## 📊 Data Structure

Form submissions are saved with the following structure:

```json
{
  "localId": "local-1737399123456",
  "syncStatus": "pending",
  "submittedAt": "2026-01-20T19:30:00.000Z",
  "requestDate": "2026-01-20",
  "completedBy": "John Doe",
  "completedByRole": "caregiver",
  "caregiverName": "Jane Smith",
  "caregiverPhone": "336-123-4567",
  "caregiverEmail": "jane@example.com",
  "socialWorkerName": "Mary Johnson",
  "requestType": "bags-of-hope",
  "childFirstName": "Emma",
  "childLastName": "Smith",
  "childAge": 6,
  "estimatedValue": 250,
  ...
}
```

## 🚀 Deployment

### Vercel (Recommended)
```bash
# First time setup
vercel login
vercel link

# Deploy to production
vercel --prod
```

### Other Static Hosts
The form is a single HTML file and can be deployed to any static hosting service:
- Netlify: Drag and drop the HTML file
- GitHub Pages: Push to `gh-pages` branch
- AWS S3: Upload as static website
- Any web server: Just upload the HTML file

## 🔄 Future Integration: Neon CRM

The form is ready for Neon CRM integration. When ready:

1. Create backend API endpoint
2. Replace `saveToLocalStorage()` with API call in form
3. Use existing data structure (already matches Neon CRM fields)
4. Implement sync for pending localStorage submissions

See `IMPLEMENTATION_SPEC.md` for detailed integration guide.

## 📞 Support

**Questions or Issues?**
- Technical: See `CLAUDE.md` and `IMPLEMENTATION_SPEC.md`
- Business Logic: Contact Rebekah Estep (rebekahe@lotcarolinas.com)

## 📝 License

© 2026 Least of These Carolinas. All rights reserved.

---

**Version:** 1.0.0
**Last Updated:** January 20, 2026
