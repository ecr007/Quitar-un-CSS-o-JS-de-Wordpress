# Quitar-un-CSS-o-JS-de-Wordpress

## 1 - Primero se procede a ver el listado de asset, estos se pueden buscar directamente WP para saber cuales son sus identificadores.

```php
// Para agregar un css a la cola
wp_enqueue_style( string $handle, string $src = '', array $deps = array(), string|bool|null $ver = false, string $media = 'all' );

// Para agregar un js a la cola
wp_enqueue_script( string $handle, string $src = '', array $deps = array(), string|bool|null $ver = false, bool $in_footer = false );

```

# Función para desplegar algunos, lo mejor es buscarlos directamente.

```php
function crunchify_print_scripts_styles() {

    $result = [];
    $result['scripts'] = [];
    $result['styles'] = [];

    // Print all loaded Scripts
    global $wp_scripts;
    foreach( $wp_scripts->queue as $script ) :
       	$result['scripts'][] =  [
       		'src' => $wp_scripts->registered[$script]->src,
       		'handle' => $wp_scripts->registered[$script]->handle,
       	];
    endforeach;

    // Print all loaded Styles (CSS)
    global $wp_styles;
    foreach( $wp_styles->queue as $style ) :
       	$result['styles'][] =  [
       		'src' => $wp_styles->registered[$style]->src,
       		'handle' => $wp_styles->registered[$style]->handle,
       	];
    endforeach;

    echo "<pre>";
    	print_r($result);
    echo "</pre>";
}

add_filter( 'wp_head', 'crunchify_print_scripts_styles' );

```

# Funcion para quitar estilos

```php
function get_style_before_load( $tag, $handle, $src ) {

	$request_uri = parse_url( $_SERVER['REQUEST_URI'], PHP_URL_PATH );

	// The handles of the enqueued scripts we want to defer
	$exclude_styles = array( 
		'tribe-tooltip',
		'wp-block-library',
		'wc-block-style',
		//'popupally-pro-style',
		'woocommerce-inline',
		'jqcss',
		'mrjswp',
		'mrcss',
		'tribe-common-skeleton-style',
		'vc_google_fonts_josefin_sans100100italic300300italicregularitalic600600italic700700italic',
		'sage/css',
		'js_composer_front',
		'production-css',
	);
  
	$excluded_styles_about = [
		'sage/css',
		'js_composer_front',
		'production-css',
		'wc-block-style',
		'wp-block-library',
		'jqcss',
		'tribe-common-skeleton-style',
		'tribe-tooltip',
	];

    if ($request_uri == '/' && in_array( $handle, $exclude_styles ) ) { // Home
        return '';
    }

    if (is_blog_page() && in_array( $handle, $excluded_styles_about ) ) {
        return '';
    }

    
    return $tag;
}

add_filter( 'style_loader_tag', 'get_style_before_load', 10, 3 );
```

# Function para quitar script

Lo mismo pero con las funciones

```php
function get_script_before_load( $tag, $handle, $src ) {

	$request_uri = parse_url( $_SERVER['REQUEST_URI'], PHP_URL_PATH );

	// The handles of the enqueued scripts we want to defer
	$exclude_scripts = array( 
		//'mr_tracking',
		'disqus_embed',
		'disqus_count',
		'popupally-pro-check-source',
		'opupally-pro-action-script',
		'wc-add-to-cart',
		'woocommerce',
		'wc-cart-fragments',
		'vc_woocommerce-add-to-cart-js',
		'jquery-blockui',
		//'genjs-v3-js',
	);

	$excluded_scripts_product = [
		'disqus_embed',
		'disqus_count',
		'jquery-blockui',
		'popupally-pro-check-source',
		'opupally-pro-action-script',
		'flexslider',
		'comment-reply',
	];

	$excluded_scripts_about = [
		'disqus_embed',
		'disqus_count',
		'jquery-blockui',
		'popupally-pro-check-source',
		'opupally-pro-action-script',
		'flexslider',
		'comment-reply',
		'woocommerce',
		'wc-cart-fragments',
	];

	$excluded_scripts_blog = [
		'jquery-blockui',
		'popupally-pro-check-source',
		'opupally-pro-action-script',
		'flexslider',
		'woocommerce',
		'wc-cart-fragments',
	];

	$defer_scripts = [

	];


    if ($request_uri == '/' && in_array( $handle, $exclude_scripts ) ) {
        return '';
    }
    if (strstr($request_uri, 'product') && in_array( $handle, $excluded_scripts_product ) ) {
        return '';
    }
    if (strstr($request_uri, 'about') && in_array( $handle, $excluded_scripts_about ) ) {
        return '';
    }
    if (is_blog_page() && in_array( $handle, $excluded_scripts_blog ) ) {
        return '';
    }
    if ( in_array( $handle, $defer_scripts ) ) {
        return '<script src="' . $src . '" defer="defer" type="text/javascript"></script>' . "\n";
    }
    
    return $tag;
}

add_filter( 'script_loader_tag', 'get_script_before_load', 10, 3 );

```

