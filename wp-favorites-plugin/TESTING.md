# Testing WP Favorites Plugin

This document explains how to test the WP Favorites Plugin functionality.

## Installation Testing

### 1. Install the Plugin
1. Upload the `wp-favorites-plugin` folder to `/wp-content/plugins/`
2. Activate the plugin through WordPress admin
3. Check that the custom table `wp_user_favorites` is created

### 2. Verify Database Table
```sql
-- Check if table exists
SHOW TABLES LIKE 'wp_user_favorites';

-- Check table structure
DESCRIBE wp_user_favorites;
```

## REST API Testing

### 1. Test Authentication
```bash
# Test without authentication (should fail)
curl -X POST "https://your-site.com/wp-json/wp-favorites/v1/favorite" \
  -H "Content-Type: application/json" \
  -d '{"post_id": 1}'
```

### 2. Test with Authentication
```bash
# Get nonce first
curl -X GET "https://your-site.com/wp-json/wp/v2/users/me" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"

# Test favorite endpoint
curl -X POST "https://your-site.com/wp-json/wp-favorites/v1/favorite" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -d '{"post_id": 1}'

# Test unfavorite endpoint
curl -X POST "https://your-site.com/wp-json/wp-favorites/v1/unfavorite" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -d '{"post_id": 1}'

# Test get favorites endpoint
curl -X GET "https://your-site.com/wp-json/wp-favorites/v1/favorites" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

## Frontend Testing

### 1. Add Favorite Button to Theme
Add this code to your theme's `single.php` or any template:

```php
<?php
// Display favorite button
do_action('wp_favorites_button', array(
    'post_id' => get_the_ID(),
    'show_count' => true,
    'button_text' => 'Favorite this post'
));
?>
```

### 2. Test Shortcodes
Add these shortcodes to any post or page:

```php
// Favorite button shortcode
[wp_favorites_button post_id="1" show_count="true"]

// Favorites list shortcode
[wp_favorites_list limit="5" show_date="true" show_excerpt="true"]
```

### 3. Test JavaScript Functionality
1. Open browser developer tools
2. Click the favorite button
3. Check network tab for API calls
4. Verify button state changes
5. Check for success/error messages

## Manual Testing Checklist

### Plugin Activation
- [ ] Plugin activates without errors
- [ ] Custom table is created
- [ ] No PHP errors in error log

### REST API Endpoints
- [ ] `/wp-json/wp-favorites/v1/favorite` (POST)
- [ ] `/wp-json/wp-favorites/v1/unfavorite` (POST)
- [ ] `/wp-json/wp-favorites/v1/favorites` (GET)

### Authentication
- [ ] Unauthenticated requests return 401
- [ ] Authenticated requests work properly
- [ ] Nonce validation works

### Database Operations
- [ ] Favoriting a post adds record to database
- [ ] Unfavoriting removes record from database
- [ ] Duplicate favorites are prevented
- [ ] Invalid post IDs are handled

### Frontend Functionality
- [ ] Favorite button displays correctly
- [ ] Button state changes on click
- [ ] Loading states work
- [ ] Success/error messages display
- [ ] Non-logged-in users see login link

### Shortcodes
- [ ] `[wp_favorites_button]` displays button
- [ ] `[wp_favorites_list]` displays favorites
- [ ] Shortcode attributes work correctly

### CSS/JS Assets
- [ ] CSS styles load properly
- [ ] JavaScript functionality works
- [ ] No console errors
- [ ] Responsive design works

## Automated Testing

### PHPUnit Tests
Create a `tests` directory and add test files:

```php
// tests/test-wp-favorites-plugin.php
class WP_Favorites_Plugin_Test extends WP_UnitTestCase {
    
    public function test_plugin_activation() {
        // Test table creation
        global $wpdb;
        $table_name = $wpdb->prefix . 'user_favorites';
        $this->assertTrue($wpdb->get_var("SHOW TABLES LIKE '$table_name'") === $table_name);
    }
    
    public function test_favorite_post() {
        // Test favoriting functionality
        $user_id = $this->factory->user->create();
        $post_id = $this->factory->post->create();
        
        wp_set_current_user($user_id);
        
        $request = new WP_REST_Request('POST', '/wp-favorites/v1/favorite');
        $request->set_param('post_id', $post_id);
        
        $response = rest_do_request($request);
        $this->assertEquals(200, $response->get_status());
    }
}
```

### JavaScript Tests
Use Jest or similar for JavaScript testing:

```javascript
// tests/wp-favorites.test.js
describe('WP Favorites Plugin', () => {
    test('should initialize favorite button', () => {
        document.body.innerHTML = `
            <button class="wp-favorites-button" data-post-id="1">
                <span class="wp-favorites-icon">♡</span>
                <span class="wp-favorites-text">Favorite</span>
            </button>
        `;
        
        // Initialize plugin
        $('.wp-favorites-button').wpFavorites();
        
        expect($('.wp-favorites-button').length).toBe(1);
    });
});
```

## Performance Testing

### Database Queries
Monitor database performance:

```php
// Enable query logging
define('SAVEQUERIES', true);

// Check query count
global $wpdb;
echo "Total queries: " . count($wpdb->queries);
```

### Memory Usage
Monitor memory usage during operations:

```php
echo "Memory usage: " . memory_get_usage(true) / 1024 / 1024 . " MB";
```

## Security Testing

### Input Validation
- [ ] Test with invalid post IDs
- [ ] Test with non-existent posts
- [ ] Test with negative numbers
- [ ] Test with string values

### SQL Injection
- [ ] Test with malicious input
- [ ] Verify prepared statements work
- [ ] Check for SQL errors

### XSS Prevention
- [ ] Test with script tags in input
- [ ] Verify output is properly escaped
- [ ] Check for reflected XSS

## Browser Testing

### Cross-browser Compatibility
Test in:
- [ ] Chrome
- [ ] Firefox
- [ ] Safari
- [ ] Edge
- [ ] Mobile browsers

### Responsive Design
- [ ] Desktop (1920x1080)
- [ ] Tablet (768x1024)
- [ ] Mobile (375x667)

## Error Handling

### Test Error Scenarios
- [ ] Network failures
- [ ] Database connection issues
- [ ] Invalid responses
- [ ] Timeout scenarios

### Logging
- [ ] Check error logs
- [ ] Verify debug information
- [ ] Test error reporting

## Cleanup Testing

### Plugin Deactivation
- [ ] Test plugin deactivation
- [ ] Verify no PHP errors
- [ ] Check database integrity

### Plugin Uninstall
- [ ] Test uninstall process
- [ ] Verify table removal
- [ ] Check option cleanup

## Integration Testing

### WordPress Core
- [ ] Test with different WordPress versions
- [ ] Test with different themes
- [ ] Test with other plugins

### REST API
- [ ] Test with different authentication methods
- [ ] Test with different content types
- [ ] Test rate limiting

## Reporting Issues

When reporting issues, include:
1. WordPress version
2. Plugin version
3. PHP version
4. Error messages
5. Steps to reproduce
6. Expected vs actual behavior
7. Browser/device information 