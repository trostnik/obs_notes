OkHttp - is a Http client for Android, meaning it is used to send requests and receive responses from a web.

Cache: requests can be cached using .cache for OkHttpClient. That means that we can define directory for response to be stored, and if request is made OkHttp client will take info from cache, skipping web events(which is much faster) and can be displayed. 

Interceptors are a powerful mechanism that can monitor, rewrite, and retry calls.

Application interceptors

Don’t need to worry about intermediate responses like redirects and retries.
Are always invoked once, even if the HTTP response is served from the cache.
Observe the application’s original intent. Unconcerned with OkHttp-injected headers like If-None-Match.
Permitted to short-circuit and not call Chain.proceed().
Permitted to retry and make multiple calls to Chain.proceed().
Can adjust Call timeouts using withConnectTimeout, withReadTimeout, withWriteTimeout.

Network Interceptors

Able to operate on intermediate responses like redirects and retries.
Not invoked for cached responses that short-circuit the network.
Observe the data just as it will be transmitted over the network.
Access to the Connection that carries the request.

WebSocket
interface WebSocket

A non-blocking interface to a web socket. Use the factory to create instances; usually this is OkHttpClient.

Web Socket Lifecycle
Upon normal operation each web socket progresses through a sequence of states:

Connecting: the initial state of each web socket. Messages may be enqueued but they won’t be transmitted until the web socket is open.

Open: the web socket has been accepted by the remote peer and is fully operational. Messages in either direction are enqueued for immediate transmission.

Closing: one of the peers on the web socket has initiated a graceful shutdown. The web socket will continue to transmit already-enqueued messages but will refuse to enqueue new ones.

Closed: the web socket has transmitted all of its messages and has received all messages from the peer.

Web sockets may fail due to HTTP upgrade problems, connectivity problems, or if either peer chooses to short-circuit the graceful shutdown process:

Canceled: the web socket connection failed. Messages that were successfully enqueued by either peer may not have been transmitted to the other.
Note that the state progression is independent for each peer. Arriving at a gracefully-closed state indicates that a peer has sent all of its outgoing messages and received all of its incoming messages. But it does not guarantee that the other peer will successfully receive all of its incoming messages.

Kotlin Android WebSocket Examples
WebSocket is a protocol that makes it possible to open a two-way interactive communication session between the client and the server. It provides a full-duplex communication channels over a single TCP connection.

This protocol defines an API that establishes a "socket" connection between a web browser and a server. Rather than long polling to get updates for example from a livescore app, you can use websockets to create a persistent connection between the client and the server. Thus the app auto-updates itself whenever newer updates are available.
This avoids the overhead of sending an additional HTTP request to check if there are newer updates.

Thus the client and server only need to complete a handshake, and a persistent connection can be created directly between the two, and two-way data transfer carried out.

Websocket

Websocket uses the same TCP port as HTTP, which can bypass most firewall restrictions. By default, the Websocket protocol uses port 80; when running on top of TLS, port 443 is used by default. Its protocol identifier is ws. If wss is used for encryption, the server web address is URL, such as:

ws://www.example.com/
wss://www.example.com/
Bash
Advantages of Websocket
Less control overhead: After the connection is created, when data is exchanged between the server and the client, the data packet header used for protocol control is relatively small
Stronger real-time performance: Since the protocol is full-duplex, the server can actively send data to the client at any time
Keep the connection state: Unlike HTTP, Websocket needs to create a connection first, which makes it a stateful protocol, and then part of the state information can be omitted when communicating. The HTTP request may need to carry status information in each request (such as identity authentication, etc.)
Better binary support: Websocket defines binary frames, which can handle binary content more easily than HTTP
Better compression effect: Compared with HTTP compression, Websocket can use the context of the previous content with proper extension support, and can significantly improve the compression rate when transferring similar data.

```java
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.TextView;

import okhttp3.OkHttpClient;
import okhttp3.Request;
import okhttp3.Response;
import okhttp3.WebSocket;
import okhttp3.WebSocketListener;
import okio.ByteString;

public class MainActivity extends AppCompatActivity {

    private Button buttonSend;
    private TextView textResult;
    private OkHttpClient mClient;

    private final class EchoWebSocketListener extends WebSocketListener {
        private static final int CLOSE_STATUS = 1000;
        @Override
        public void onOpen(WebSocket webSocket, Response response) {
            webSocket.send("What's up ?");
            webSocket.send(ByteString.decodeHex("abcd"));
            webSocket.close(CLOSE_STATUS, "Socket Closed !!");
        }
        @Override
        public void onMessage(WebSocket webSocket, String message) {
            print("Receive Message: " + message);
        }
        @Override
        public void onMessage(WebSocket webSocket, ByteString bytes) {
            print("Receive Bytes : " + bytes.hex());
        }
        @Override
        public void onClosing(WebSocket webSocket, int code, String reason) {
            webSocket.close(CLOSE_STATUS, null);
            print("Closing Socket : " + code + " / " + reason);
        }
        @Override
        public void onFailure(WebSocket webSocket, Throwable throwable, Response response) {
            print("Error : " + throwable.getMessage());
        }
    }
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        buttonSend = (Button) findViewById(R.id.buttonSend);
        textResult = (TextView) findViewById(R.id.textResult);
        mClient = new OkHttpClient();
        buttonSend.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                start();
            }
        });
    }
    private void start() {
        Request request = new Request.Builder().url("ws://echo.websocket.org").build();
        EchoWebSocketListener listener = new EchoWebSocketListener();
        WebSocket webSocket = mClient.newWebSocket(request, listener);
        mClient.dispatcher().executorService().shutdown();
    }
    private void print(final String message) {
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                textResult.setText(textResult.getText().toString() + "\n" + message);
            }
        });
    }
}


```