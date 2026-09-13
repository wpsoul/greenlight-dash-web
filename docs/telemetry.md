# Install statistics (telemetry)

Once a day the server POSTs one JSON document to `GL_TELEMETRY_URL`
(default `https://greenlightdash.pro/wp-json/greenlight/v1/server-ping`):

```json
{"instance": "3f9c…(24 hex, sha256 of the domain)", "version": "10.9.2", "profile": "full",
 "license": "free", "platform": "linux-x86_64", "ts": 1789300000}
```

Nothing else: no content, hostnames, tokens or IP-derived data is added by the server
(your web server sees the connection's IP like any HTTP request). Switch off with
`GL_TELEMETRY=0`. Failures are silent and never affect the app.

## Receiving side (WordPress, greenlightdash.pro)

A minimal REST route that stores the last ping per instance. Drop into a small plugin or
the WPLS companion plugin:

```php
add_action('rest_api_init', function () {
    register_rest_route('greenlight/v1', '/server-ping', [
        'methods'  => 'POST',
        'permission_callback' => '__return_true',
        'callback' => function (WP_REST_Request $r) {
            $b = $r->get_json_params();
            if (!is_array($b) || empty($b['instance']) || !preg_match('/^[0-9a-f]{24}$/', $b['instance'])) {
                return new WP_REST_Response(['ok' => false], 400);
            }
            $pings = get_option('gl_server_pings', []);
            $pings[$b['instance']] = [
                'version'  => substr(sanitize_text_field($b['version'] ?? ''), 0, 32),
                'profile'  => substr(sanitize_text_field($b['profile'] ?? ''), 0, 16),
                'license'  => substr(sanitize_text_field($b['license'] ?? ''), 0, 16),
                'platform' => substr(sanitize_text_field($b['platform'] ?? ''), 0, 32),
                'seen'     => time(),
            ];
            update_option('gl_server_pings', $pings, false);
            return ['ok' => true];
        },
    ]);
});
```

Count active installations as the entries whose `seen` is within the last 7 days.
