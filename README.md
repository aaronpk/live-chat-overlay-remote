# Live Chat Overlay Remote Window

The remote window for the [Live Chat Overlay](https://github.com/aaronpk/live-chat-overlay) browser extension.

## Setup

You'll need to run this on nginx with the [nginx push-stream module](https://github.com/wandenberg/nginx-push-stream-module). Follow the [installation instructions](https://github.com/wandenberg/nginx-push-stream-module#installation-) to get the module running in your nginx before continuing below.

### Configuring nginx

Once the push-stream module is installed, you can create a server block for this website.

Download this entire repo into a folder such as `/web/sites/live-chat-overlay-remote`, and configure an nginx server block to point to the `public` folder in the repo. Enable the push-stream module publisher and subscriber using the configuration below.

```
server {
  listen       *:443 ssl http2;
  server_name  chat.example.com;

  ssl_certificate /etc/letsencrypt/live/chat.example.com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/chat.example.copm/privkey.pem;

  root /web/sites/live-chat-overlay-remote/public;
  index index.html;

  location /overlay/pub {
    add_header 'Access-Control-Allow-Origin' '*';
    push_stream_publisher admin;
    push_stream_channels_path    $arg_id;
  }

  location /overlay/sub {
    add_header 'Access-Control-Allow-Origin' '*';
    push_stream_subscriber eventsource;
    push_stream_channels_path    $arg_id;
    push_stream_message_template                "{\"id\":~id~,\"channel\":\"~channel~\",\"text\":~text~}";
    push_stream_ping_message_interval           10s;
  }

}
```

That's it! No server-side code environment is needed as the project is static HTML and JavaScript.

### Browser Extension Configuration

In your Live Chat Overlay extension, set the remote URL to the path of the "overlay" folder in this project, for example: `https://chat.example.com/overlay/`. The extension will use that full URL as the main browser source URL, and the extension will expect the `pub` path to exist where it will be pushing the chat messages to.

