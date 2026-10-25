---
name: wordpress-modern
description: Modern WordPress 6.8+ features including asset loading optimization, speculative loading, enqueue strategies, settings API, AJAX handlers, activation hooks, and performance best practices. Use when optimizing performance, implementing settings pages, or using modern WP 6.8 APIs.
origin: mikkelkrogsholm/wp-plugins (via mattnigh/skills_collection)
---

# WordPress Modern Features Skill

Modern WordPress 6.8+ performance features and optimization patterns.

## Modern Asset Loading (WordPress 6.8+)

WordPress 6.8 introduces on-demand block asset loading for better performance.

### On-Demand Loading

```php
// Enable on-demand loading for your block
function my_block_should_load_assets( $should_load, $block ) {
	if ( 'my-plugin/my-block' === $block['blockName'] ) {
		// Only load if block is actually used on page
		return true;
	}
	return $should_load;
}
add_filter( 'should_load_block_assets_on_demand', 'my_block_should_load_assets', 10, 2 );
```

### Speculative Loading Filters (WordPress 6.8)

Improve perceived performance with speculative loading:

```php
// Exclude specific paths from speculation
function my_exclude_speculation_paths( $excluded_paths ) {
	$excluded_paths[] = '/checkout/*';
	$excluded_paths[] = '/my-account/*';
	return $excluded_paths;
}
add_filter( 'wp_speculation_rules_href_exclude_paths', 'my_exclude_speculation_paths' );

// Configure speculation rules
function my_speculation_config( $config ) {
	$config['prerender'] = array(
		'eagerness' => 'moderate', // conservative, moderate, eager
	);
	return $config;
}
add_filter( 'wp_speculation_rules_configuration', 'my_speculation_config' );

// Add custom speculation rules
function my_custom_speculation_rules( $rules ) {
	$rules[] = array(
		'source'    => 'list',
		'urls'      => array( '/priority-page/', '/important-page/' ),
		'eagerness' => 'eager',
	);
	return $rules;
}
add_filter( 'wp_load_speculation_rules', 'my_custom_speculation_rules' );
```

**Security Note**: Never prerender authenticated pages or forms with CSRF tokens. Always exclude checkout, login, and account pages from speculation.

### Conditional Script Loading

```php
function my_conditional_enqueue() {
	// Only load on specific pages
	if ( is_singular( 'product' ) ) {
		wp_enqueue_script(
			'my-product-script',
			plugin_dir_url( __FILE__ ) . 'js/product.js',
			array(),
			'1.0.0',
			true
		);
	}
}
add_action( 'wp_enqueue_scripts', 'my_conditional_enqueue' );
```

## Enqueue Scripts and Styles

```php
function my_enqueue_scripts() {
	// CSS
	wp_enqueue_style(
		'my-plugin-style',
		plugin_dir_url( __FILE__ ) . 'css/style.css',
		array(),
		'1.0.0'
	);

	// JavaScript
	wp_enqueue_script(
		'my-plugin-script',
		plugin_dir_url( __FILE__ ) . 'js/script.js',
		array( 'jquery' ), // Dependencies
		'1.0.0',
		true // Load in footer
	);

	// Pass data to JavaScript
	wp_localize_script( 'my-plugin-script', 'myPluginData', array(
		'ajaxUrl' => admin_url( 'admin-ajax.php' ),
		'nonce'   => wp_create_nonce( 'my_ajax_nonce' ),
	) );
}
add_action( 'wp_enqueue_scripts', 'my_enqueue_scripts' );
```

## Settings API

```php
// Register settings
function my_register_settings() {
	register_setting( 'my_options_group', 'my_option_name' );

	add_settings_section(
		'my_section_id',
		'Section Title',
		'my_section_callback',
		'my-settings-page'
	);

	add_settings_field(
		'my_field_id',
		'Field Label',
		'my_field_callback',
		'my-settings-page',
		'my_section_id'
	);
}
add_action( 'admin_init', 'my_register_settings' );

function my_section_callback() {
	echo '<p>Section description</p>';
}

function my_field_callback() {
	$value = get_option( 'my_option_name' );
	echo '<input type="text" name="my_option_name" value="' . esc_attr( $value ) . '">';
}

// Settings page
function my_settings_page() {
	?>
	<div class="wrap">
		<h1><?php echo esc_html( get_admin_page_title() ); ?></h1>
		<form method="post" action="options.php">
			<?php
			settings_fields( 'my_options_group' );
			do_settings_sections( 'my-settings-page' );
			submit_button();
			?>
		</form>
	</div>
	<?php
}
```

## AJAX Handlers

```php
// For logged-in users
add_action( 'wp_ajax_my_action', 'my_ajax_handler' );

// For non-logged-in users
add_action( 'wp_ajax_nopriv_my_action', 'my_ajax_handler' );

function my_ajax_handler() {
	// Verify nonce
	check_ajax_referer( 'my_ajax_nonce', 'nonce' );

	// Verify capability
	if ( ! current_user_can( 'edit_posts' ) ) {
		wp_send_json_error( array( 'message' => 'Insufficient permissions' ) );
	}

	// Process request
	$data = sanitize_text_field( $_POST['data'] );

	// Return response
	wp_send_json_success( array(
		'message' => 'Success',
		'data'    => $data,
	) );
}
```

## Activation and Deactivation Hooks

```php
// Activation
register_activation_hook( __FILE__, 'my_plugin_activate' );
function my_plugin_activate() {
	flush_rewrite_rules();

	if ( ! get_option( 'my_plugin_version' ) ) {
		update_option( 'my_plugin_version', '1.0.0' );
		update_option( 'my_plugin_settings', array(
			'enabled' => true,
		) );
	}
}

// Deactivation
register_deactivation_hook( __FILE__, 'my_plugin_deactivate' );
function my_plugin_deactivate() {
	flush_rewrite_rules();
	delete_transient( 'my_plugin_cache' );
}
```

## WordPress 6.8 Specific Features

### Image Resolution Control

```php
// Register custom image sizes for cover blocks
add_image_size( 'cover-high-res', 2400, 1600, true );
add_image_size( 'cover-medium', 1200, 800, true );

// Make available to editor
function my_custom_image_sizes( $sizes ) {
	return array_merge( $sizes, array(
		'cover-high-res' => __( 'Cover High Resolution' ),
		'cover-medium'   => __( 'Cover Medium' ),
	) );
}
add_filter( 'image_size_names_choose', 'my_custom_image_sizes' );
```

### Query Loop Enhancements

```php
// Sort pages by menu order in Query Loop
// This is now natively supported in block settings (WordPress 6.8)
// Can also ignore sticky posts for custom queries
```

## Performance Best Practices

### Caching with Transients

```php
function my_get_expensive_data() {
	$cache_key = 'my_expensive_data';
	$cached    = get_transient( $cache_key );

	if ( false !== $cached ) {
		return $cached;
	}

	$data = perform_expensive_query();
	set_transient( $cache_key, $data, HOUR_IN_SECONDS );

	return $data;
}
```

### Object Caching

```php
function my_get_cached_data( $key ) {
	$cache_group = 'my_plugin';
	$cached      = wp_cache_get( $key, $cache_group );

	if ( false !== $cached ) {
		return $cached;
	}

	$data = get_data_from_source( $key );
	wp_cache_set( $key, $data, $cache_group, 3600 );

	return $data;
}
```

### Lazy Loading Assets

```php
function my_lazy_load_scripts() {
	if ( ! is_admin() && is_singular( 'post' ) ) {
		wp_enqueue_script(
			'my-post-specific-script',
			plugin_dir_url( __FILE__ ) . 'js/post.js',
			array(),
			'1.0.0',
			true
		);
	}
}
add_action( 'wp_enqueue_scripts', 'my_lazy_load_scripts' );
```

### Database Query Optimization

```php
// Bad: Multiple queries in loop
foreach ( $post_ids as $post_id ) {
	$meta = get_post_meta( $post_id, 'my_key', true );
}

// Good: Single query with update_meta_cache
$posts = get_posts( array( 'post__in' => $post_ids ) );
update_meta_cache( 'post', $post_ids );
foreach ( $posts as $post ) {
	$meta = get_post_meta( $post->ID, 'my_key', true ); // Now cached
}
```

## Version Compatibility

```php
function my_plugin_supports_wp_68_features() {
	global $wp_version;
	return version_compare( $wp_version, '6.8', '>=' );
}

if ( my_plugin_supports_wp_68_features() ) {
	add_filter( 'wp_speculation_rules_configuration', 'my_speculation_config' );
}
```

## WordPress 6.8 Migration Notes

**No breaking changes**, but be aware:
- On-demand asset loading is opt-in — existing enqueues unaffected
- Speculation rules are new — test thoroughly; never prerender auth pages or forms with CSRF tokens

**Recommended audits before launch:**
- Implement on-demand loading for block assets
- Configure speculation rules for tour/destination pages (exclude booking flow)
- Audit enqueue strategies in child theme `functions.php`
- Test with `SCRIPT_DEBUG` enabled

## Best Practices Summary

1. Only enqueue scripts/styles where actually needed (`is_singular`, `is_page`, etc.)
2. Use transients and object cache for expensive third-party plugin or meta queries
3. Batch `get_post_meta` calls — use `update_meta_cache` before loops
4. Never speculate/prerender booking forms or any page with a nonce
5. Gracefully degrade for WP < 6.8 with version checks
