# Quick Guide: Navbar Logo Customization

## ✅ What's Been Set Up

Your Keycloak instance now has custom themes that will replace logos in:
1. **Login pages** - The main login, registration, and password reset screens
2. **Admin console navbar** - The top-left logo when logged into /admin
3. **Account console navbar** - The logo in the user account management interface

## 🚀 How to Enable

### 1. Access Admin Console
```
http://localhost:8080/admin
Login: admin / admin (or your credentials)
```

### 2. Configure Themes

Navigate to: **Realm settings** → **Themes** tab

Then set these dropdowns:

| Theme Type | Select | Effect |
|------------|--------|---------|
| **Login theme** | `custom` | Changes login page logo |
| **Admin console theme** | `custom` | Changes navbar logo in admin console |
| **Account theme** | `custom` | Changes navbar logo in account console |

Click **Save** after each selection.

### 3. See Your Changes

1. **Login pages**: Logout and go to http://localhost:8080 - you'll see your Logoipsum logo
2. **Admin console**: After selecting the admin theme, refresh the page with **Ctrl+Shift+R** (or **Cmd+Shift+R** on Mac)
3. **Account console**: Go to http://localhost:8080/realms/master/account - your logo will be in the navbar

## 🎨 How It Works

### Login Pages
- Direct image replacement via theme structure
- Logo files: `themes-custom/custom/login/resources/img/keycloak-logo-text.svg`

### Admin & Account Console Navbar
- CSS override using `content: url()` technique
- No need to rebuild the React application!
- CSS files automatically replace logo images in the navbar
- Works across all pages in the admin console

### Technical Details
The admin console theme includes custom CSS:
```css
/* From themes-custom/custom/admin/resources/css/custom-logo.css */
.pf-v5-c-page__header-brand-link img {
    content: url('../logo.svg') !important;
}
```

This CSS rule intercepts the logo rendering and replaces it with your custom logo.

## 🔍 Troubleshooting

### Logo Not Showing
1. **Clear browser cache**: Hard refresh with Ctrl+Shift+R (or Cmd+Shift+R on Mac)
2. **Check theme is selected**: Verify "custom" is selected for the appropriate theme type
3. **Try incognito/private window**: This ensures no caching issues

### Admin console logo still default
1. Make sure you selected "custom" for **Admin console theme** (not just Login theme)
2. Hard refresh the page after selecting the theme
3. Check browser developer console for any CSS errors

### Changes not appearing after updating logo files
```bash
# For development (with volume mount):
docker-compose restart keycloak

# For production (themes baked into image):
docker-compose build keycloak && docker-compose up -d
```

## 📁 File Locations

Your custom logo files are in:
```
themes-custom/custom/
├── admin/resources/
│   ├── css/custom-logo.css    # CSS that overrides navbar logo
│   ├── logo.svg                # Your custom logo
│   └── img/
│       ├── favicon.svg
│       ├── icon.svg
│       └── logo.svg
├── account/resources/
│   ├── css/custom-logo.css
│   └── img/logo.svg
└── login/resources/img/
    ├── keycloak-logo-text.svg  # Your custom logo
    └── keycloak-logo-text.png
```

## 🔄 Updating Logos

To change logos in the future:

1. Replace the logo files in `themes-custom/custom/*/resources/`
2. For **development** (with volume mount):
   ```bash
   docker-compose restart keycloak
   ```
3. For **production** (baked into image):
   ```bash
   docker-compose build keycloak && docker-compose up -d
   ```
4. Hard refresh your browser

## ℹ️ Additional Info

- See `THEME_CUSTOMIZATION.md` for comprehensive documentation
- The setup works with both local development and production deployment
- Themes are automatically included in the Docker image for Coolify deployment
