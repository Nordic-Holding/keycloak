# Keycloak Theme Customization Guide

## Overview

This guide explains how to customize Keycloak logos and themes for your deployment.

## What Was Changed

### 1. Custom Theme Structure Created

A new `themes-custom/` directory has been created with the following structure:

```
themes-custom/
└── custom/
    ├── admin/
    │   ├── resources/
    │   │   ├── css/
    │   │   │   └── custom-logo.css     # CSS to override navbar logo
    │   │   ├── img/
    │   │   │   ├── favicon.svg
    │   │   │   ├── icon.svg
    │   │   │   └── logo.svg
    │   │   └── logo.svg
    │   └── theme.properties
    ├── account/
    │   ├── resources/
    │   │   ├── css/
    │   │   │   └── custom-logo.css     # CSS to override navbar logo
    │   │   └── img/
    │   │       └── logo.svg
    │   └── theme.properties
    └── login/
        ├── resources/
        │   └── img/
        │       ├── keycloak-logo-text.svg
        │       └── keycloak-logo-text.png
        └── theme.properties
```

### 2. Docker Configuration Updated

- **Dockerfile.keycloak**: Updated to copy custom themes into the container
- **docker-compose.override.yml**: Added volume mount for easier development

### 3. Custom Logos

Your custom "Logoipsum" logos have been copied to the custom theme directories.

## How to Use the Custom Theme

### Step 1: Rebuild and Restart (Already Done)

The containers have been rebuilt with the custom themes. If you make changes to the logos in the future:

```bash
# For production (bakes themes into image):
docker-compose down
docker-compose build keycloak
docker-compose up -d

# For development (uses volume mount - just restart):
docker-compose restart keycloak
```

### Step 2: Configure Keycloak to Use Custom Theme

1. **Login to Keycloak Admin Console**
   - Go to http://localhost:8080/admin
   - Login with your admin credentials (default: admin/admin)

2. **Configure Login Theme for Master Realm**
   - In the left sidebar, select **Realm settings**
   - Click on the **Themes** tab
   - Under **Login theme**, select **custom** from the dropdown
   - Click **Save**

3. **Configure Login Theme for Other Realms**
   - Switch to your target realm (top-left dropdown)
   - Repeat the same process for each realm

4. **Test the Changes**
   - Logout from the admin console
   - Go to http://localhost:8080
   - You should see your custom logo on the login page

### Step 3: Configure Admin Console Theme

The custom theme now includes CSS overrides that will automatically replace the logo in the admin console navbar!

1. **Login to Keycloak Admin Console**
   - Go to http://localhost:8080/admin
   - Login with your admin credentials

2. **Configure Admin Theme**
   - Click on **Realm settings** in the left sidebar
   - Click on the **Themes** tab
   - Under **Admin console theme**, select **custom** from the dropdown
   - Click **Save**

3. **Configure Account Console Theme (Optional)**
   - Under **Account theme**, select **custom** from the dropdown
   - Click **Save**

4. **Test the Changes**
   - Refresh the page (hard refresh with Ctrl+Shift+R or Cmd+Shift+R)
   - The logo in the top navbar should now show your custom logo!

**Note**: The admin and account console logos are overridden using CSS that replaces the image content. This means:
- ✅ No need to rebuild the React application
- ✅ Works across all admin console pages
- ✅ Updates instantly when you refresh
- ⚠️ Make sure to clear browser cache if logo doesn't update immediately

## Directory Structure Explained

### themes-custom/custom/login/
This theme customizes the login, registration, and password reset pages.

**Key files**:
- `theme.properties`: Inherits from keycloak.v2 parent theme
- `resources/img/keycloak-logo-text.svg`: The logo shown on login pages
- `resources/img/keycloak-logo-text.png`: Fallback PNG version

### themes-custom/custom/admin/
This theme customizes the admin console, including the navbar logo.

**Key files**:
- `theme.properties`: Inherits from keycloak.v2 parent theme, references custom CSS
- `resources/css/custom-logo.css`: CSS that overrides the navbar logo using `content: url()` technique
- `resources/logo.svg`: Your custom logo file
- `resources/img/logo.svg`, `favicon.svg`, `icon.svg`: Logo variants for different contexts

**How it works**: The CSS file uses the `content: url()` property to replace any logo images in the admin console navbar with your custom logo. This approach works without needing to rebuild the React application.

### themes-custom/custom/account/
This theme customizes the account console (user self-service portal).

**Key files**:
- `theme.properties`: Inherits from keycloak.v3 parent theme
- `resources/css/custom-logo.css`: CSS overrides for logos
- `resources/img/logo.svg`: Your custom logo

## Making Further Customizations

### To Change Logos

1. Replace the logo files in `themes-custom/custom/*/resources/img/`
2. For development, just restart: `docker-compose restart keycloak`
3. For production, rebuild: `docker-compose build keycloak && docker-compose up -d`

### To Customize Styles

1. Create a `resources/css/` directory in your theme
2. Add custom CSS files
3. Reference them in `theme.properties`:
   ```properties
   styles=css/custom.css
   ```

### To Customize Templates

1. Create a directory matching the template type (e.g., `login/`)
2. Copy the original template from the parent theme
3. Modify as needed
4. Keycloak will use your custom template instead of the parent

## Troubleshooting

### Logo Not Showing Up

1. **Clear browser cache**: Hard refresh with Ctrl+Shift+R (or Cmd+Shift+R on Mac)
2. **Check theme is selected**: Go to Realm settings > Themes and verify "custom" is selected
3. **Verify files exist in container**:
   ```bash
   docker exec keycloak ls -la /opt/keycloak/themes/custom/login/resources/img/
   ```

### Container Won't Start

Check the logs:
```bash
docker-compose logs keycloak
```

### Volume Mount Issues

If using the volume mount for development and changes aren't reflected:
1. Ensure the path in docker-compose.override.yml is correct
2. Restart the container: `docker-compose restart keycloak`
3. Check file permissions

## Production Deployment

For production (e.g., Coolify), the themes are already baked into the Docker image via the Dockerfile COPY command. The volume mount in docker-compose.override.yml is only for local development and won't be used in production.

## Additional Resources

- [Keycloak Themes Documentation](https://www.keycloak.org/docs/latest/server_development/#_themes)
- [Keycloak Server Development Guide](https://www.keycloak.org/docs/latest/server_development/)

## Summary

✅ Custom theme structure created for login, admin, and account consoles
✅ Custom logos copied to all theme directories
✅ CSS overrides created to replace navbar logos in admin/account consoles
✅ Docker configuration updated with theme inclusion
✅ Containers rebuilt and running with all custom themes

**What logos are customized**:
- 🔑 **Login pages**: Logo appears on login, registration, and password reset pages
- 📊 **Admin console navbar**: Logo in the top-left of the admin interface (after theme is selected)
- 👤 **Account console navbar**: Logo in the user account management interface (after theme is selected)

**Next Steps**:
1. Configure Login theme (Realm settings → Themes → Login theme → custom)
2. Configure Admin console theme (Realm settings → Themes → Admin console theme → custom)
3. Configure Account theme (Realm settings → Themes → Account theme → custom)
4. Hard refresh browser (Ctrl+Shift+R or Cmd+Shift+R) to see changes
