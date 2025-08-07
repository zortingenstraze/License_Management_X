# BALKAy License Manager WordPress Plugin

Always follow these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

**CRITICAL SETUP REQUIREMENTS:**
- This is a WordPress plugin that REQUIRES a WordPress installation to function
- Plugin CANNOT be tested standalone - WordPress environment is mandatory
- Plugin files are located in the repository root (license-manager.php is the main file)

### WordPress Environment Setup
- Set up WordPress 6.4+ with PHP 8.1+ and MySQL 8.0+
- Use Docker Compose for quick setup:
```yaml
version: '3.8'
services:
  wordpress:
    image: wordpress:6.4-php8.1-apache
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - /path/to/repo:/var/www/html/wp-content/plugins/license-manager
    depends_on:
      - db
  db:
    image: mysql:8.0
    environment:
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
      MYSQL_ROOT_PASSWORD: somewordpress
```

### Plugin Installation and Activation
- Copy plugin files to `/wp-content/plugins/license-manager/` 
- **REQUIRED**: Activate plugin via WordPress Admin → Plugins → BALKAy Lisans Yöneticisi → Activate
- **REQUIRED**: Set permalinks to "Post name" (not Plain) in Settings → Permalinks for API endpoints to work
- Plugin setup takes **under 5 seconds** after WordPress is running

### Core Functionality Testing
**VALIDATION SCENARIOS - ALWAYS TEST THESE:**
1. **Plugin Activation**: Verify "Plugin activated" message appears
2. **Admin Menu**: Confirm "Lisans Yöneticisi" menu appears with 7 submenus
3. **Dashboard Access**: Visit `wp-admin/admin.php?page=license-manager` - should show statistics dashboard
4. **API Endpoints**: Test all REST API endpoints (requires permalinks set to "Post name"):
   - `POST /wp-json/balkay-license/v1/validate_license`
   - `POST /wp-json/balkay-license/v1/validate` 
   - `GET /wp-json/balkay-license/v1/license_info`
   - `POST /wp-json/balkay-license/v1/check_status`
   - `POST /api/validate_license`

### API Testing Commands
```bash
# Test primary validation endpoint
curl -X POST "http://localhost:8080/wp-json/balkay-license/v1/validate_license" \
  -H "Content-Type: application/json" \
  -d '{"license_key": "TEST-KEY", "domain": "localhost"}'

# Expected response: {"status":"invalid","license_type":"","expires_on":"","user_limit":0,"modules":[],"message":"Geçersiz lisans anahtarı","handler":"direct_bypass"}

# Test GET endpoint
curl "http://localhost:8080/wp-json/balkay-license/v1/license_info?license_key=TEST-KEY"

# Test alternative API
curl -X POST "http://localhost:8080/api/validate_license" \
  -H "Content-Type: application/json" \
  -d '{"license_key": "TEST-KEY", "domain": "localhost"}'
```

## Build and Test Process

### Code Validation (No Build Required)
- **PHP Syntax Check**: `php -l license-manager.php` - takes <1 second
- **All Include Files**: `find includes/ -name "*.php" -exec php -l {} \;` - takes <3 seconds
- **No compilation or build steps required** - this is pure PHP

### Built-in Test Suite
- **Access**: Visit `wp-admin/admin.php?page=license-manager&run_test=1` (requires admin login)
- **Tests included**: Database structure, module management, API endpoints, access control
- **Test execution time**: <2 seconds
- **Manual testing only** - no CLI test runner available

### WordPress Admin Interface Testing
**Complete admin workflow validation:**
1. Dashboard: Statistics display (all 0 for fresh install)
2. Customers: Customer management interface (`admin.php?page=license-manager-customers`)
3. Licenses: License management interface (`admin.php?page=license-manager-licenses`)
4. Settings: Configuration options (`admin.php?page=license-manager-settings`)
5. All interfaces respond in <1 second

## Performance and Timing

### Response Times (Docker environment)
- **WordPress startup**: ~30 seconds for initial container setup
- **Plugin activation**: <5 seconds
- **API response time**: <1 second per request
- **Admin page loads**: <1 second
- **Database operations**: <1 second

### Timeout Recommendations
- **Docker setup**: Set 5 minutes timeout for initial container downloads
- **WordPress installation**: Set 2 minutes timeout
- **Plugin operations**: Default 30 seconds timeout sufficient
- **API testing**: Default timeouts sufficient (responses are immediate)

## Common Development Tasks

### File Structure Overview
```
├── license-manager.php          # Main plugin file
├── includes/                    # Core PHP classes
│   ├── class-license-manager-admin.php
│   ├── class-license-manager-api.php
│   ├── class-license-manager-customer.php
│   ├── class-license-manager-database.php
│   ├── class-license-manager-license.php
│   ├── class-license-manager-modules.php
│   └── test-suite.php          # Built-in testing
├── api/endpoints/               # Additional API endpoints
├── assets/css/                  # Stylesheets
├── assets/js/                   # JavaScript files
└── _ClientSide_Files/          # Client-side components
```

### Configuration Requirements
- **WordPress Version**: 5.0+ (tested with 6.4.3)
- **PHP Version**: 7.4+ (tested with 8.1)
- **MySQL Version**: 5.6+ (tested with 8.0)
- **Permalink Structure**: MUST be set to "Post name" or custom (not Plain)

### Database Initialization
- **Automatic setup**: Plugin creates all necessary tables and default data on activation
- **No manual database setup required**
- **Tables created**: Uses WordPress custom post types (`lm_customer`, `lm_license`, `lm_license_package`)

### API Integration Notes
- **REST API Namespace**: `balkay-license/v1`
- **Response Format**: JSON with status, license_type, expires_on, user_limit, modules, message
- **Error Handling**: Returns appropriate HTTP status codes with descriptive messages
- **Domain Validation**: Built-in domain checking for license security

### Troubleshooting Common Issues
- **API returns 404**: Check permalink structure - must not be "Plain"
- **Plugin menu not visible**: Ensure plugin is activated and user has admin privileges  
- **Database errors**: Check WordPress database configuration
- **PHP errors**: Verify PHP 7.4+ compatibility and WordPress requirements

## Validation Checklist
**Before making changes, always verify:**
- [ ] WordPress environment is running
- [ ] Plugin is activated
- [ ] Permalinks set to "Post name"
- [ ] Admin menu "Lisans Yöneticisi" is visible
- [ ] API endpoints return JSON responses
- [ ] Dashboard shows statistics (may be 0 for fresh install)

**After making changes, always test:**
- [ ] PHP syntax validation passes
- [ ] Plugin still activates without errors
- [ ] Admin interface still accessible
- [ ] API endpoints still functional
- [ ] No WordPress fatal errors in debug log

## Integration Context
This plugin provides license management for the Insurance CRM system. The API endpoints exactly match the specifications in `LICENSE_SYSTEM_README.md` for seamless integration with the main CRM application.

Fixes #4.