import java.io.IOException;
import java.net.ServerSocket;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;


/**
 * ProxyServer listens to a port and launches new thread per request
 * 
 * @author hwong3
 */
public class ProxyServer {

	/**
	 * Proxy server instance
	 */
	protected ServerSocket server;
	
	/**
	 * Executors.newCachedThreadPool() launches new thread as needed and usage idle threads
	 * idle threads will die after being unused for 60 seconds
	 */
	protected ExecutorService executor;
	
	/**
	 * Default HTTP request port
	 */
	protected static int LISTEN_PORT = 8080;

	/**
	 * Create new ProxyServer instance and listen to a port
	 * 
	 * @param port	ProxyServer listening port
	 */
	public ProxyServer (int port) {
		
		executor = Executors.newCachedThreadPool();
		try { server = new ServerSocket(port); }
		catch (IOException e) {	}
	}

	/**
	 * Create new socket and request handler object on each request
	 * 
	 */
	public void accept() {

		while (true) {
			try { executor.execute(new RequestHandler(server.accept())); }
			catch (IOException e) {	}			
		}
	}

	/**
	 * Main method to fire up proxy server and launch request handlers
	 * 
	 */
	public static void main(String[] args) {

		System.out.println("ProxyServer is listening to port "+LISTEN_PORT);
		ProxyServer proxy = new ProxyServer(LISTEN_PORT);
		proxy.accept();
	}

}
