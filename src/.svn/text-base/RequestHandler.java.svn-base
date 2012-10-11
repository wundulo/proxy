import java.io.DataInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.Socket;
import java.net.SocketException;
import java.net.UnknownHostException;
import java.util.HashMap;
import java.util.StringTokenizer;


/**
 * RequestHandler processes each client's request through proxy server, 
 * then sends to remote server and write back to client.
 * 
 * @author hwong3
 *
 */
public class RequestHandler implements Runnable {

	/**
	 * Client input stream for reading request
	 */
	protected DataInputStream clientInputStream;
	
	
	/**
	 * Client output stream for rendering response
	 */
	protected OutputStream clientOutputStream;

	/**
	 * Remote output stream to send in client's request
	 */
	protected OutputStream remoteOutputStream;
	
	/**
	 * Remote input stream to read back response to client
	 */
	protected InputStream remoteInputStream;

	/**
	 * Client socket object
	 */
	protected Socket clientSocket;
	
	/**
	 * Remote socket object 
	 */
	protected Socket remoteSocket;

	/**
	 * Client request type (Only "GET" or "POST" are handled)
	 */
	protected String requestType;
	
	/**
	 * Client request url (e.g. http://www.google.com) 
	 */
	protected String url;
	
	/**
	 * Client request uri parsed from url (e.g. /index.html) 
	 */
	protected String uri;
	
	/**
	 * Client request version (e.g. HTTP/1.1) 
	 */
	protected String httpVersion;

	/**
	 * Data structure to hold all client request handers (e.g. proxy-connection: keep-alive)
	 */
	protected HashMap<String, String> header;

	/**
	 * End of line character
	 */
	static String endOfLine = "\r\n";

	/**
	 * Create a RequestHandler instance with clientSocket object
	 * 
	 * @param clientSocket	Client socket object
	 */
	public RequestHandler(Socket clientSocket) {
		header = new HashMap<String, String>();
		this.clientSocket = clientSocket;
	}

	/** 
	 * When instance is created, open client/remote streams then 
	 * proceed with the following 3 tasks:<br>
	 * 
	 * 1) get request from client<br>
	 * 2) forward request to remote host<br>
	 * 3) read response from remote back to client<br>
	 * 
	 * Close client/remote streams when finished.<br>
	 * 
	 * @see java.lang.Runnable#run()
	 */
	public void run() {

		try {

			clientInputStream = new DataInputStream(clientSocket.getInputStream());
			clientOutputStream = clientSocket.getOutputStream();

			// step 1) get request from client
			clientToProxy();

			// step 2) forward request to remote host
			proxyToRemote();

			// step 3) read response from remote back to client
			remoteToClient();

			System.out.println();

			if(remoteOutputStream != null) remoteOutputStream.close();
			if(remoteInputStream != null) remoteInputStream.close();
			if(remoteSocket != null) remoteSocket.close();


			if(clientOutputStream != null) clientOutputStream.close();
			if(clientInputStream != null) clientInputStream.close();
			if(clientSocket != null) clientSocket.close();

		} catch (IOException e) { }
	}

	/**
	 * Receive and pre-process client's request headers before redirecting to remote server
	 * 
	 */
	@SuppressWarnings("deprecation")
	private void clientToProxy() {

		String line, key, value;
		StringTokenizer tokens;

		try {

			// HTTP Command
			if(( line = clientInputStream.readLine()) != null) {
				tokens = new StringTokenizer(line);
				requestType = tokens.nextToken();
				url = tokens.nextToken();
				httpVersion = tokens.nextToken();
			}

			// Header Info
			while((line = clientInputStream.readLine()) != null) {
				// check for empty line
				if(line.trim().length() == 0) break;

				// tokenize every header as key and value pair
				tokens = new StringTokenizer(line);
				key = tokens.nextToken(":");
				value = line.replaceAll(key, "").replace(": ", "");
				header.put(key.toLowerCase(), value);
			}

			stripUnwantedHeaders();
			getUri();
		} 
		catch (UnknownHostException e) { return; } 
		catch (SocketException e){ return; } 
		catch (IOException e) { return;} 
	}

	/**
	 * Sending pre-processed client request to remote server
	 * 
	 */
	private void proxyToRemote() {

		try{
			if(header.get("host") == null) return;
			if(!requestType.startsWith("GET") && !requestType.startsWith("POST")) 
				return;

			remoteSocket = new Socket(header.get("host"), 80);
			remoteOutputStream = remoteSocket.getOutputStream();

			// make sure streams are still open
			checkRemoteStreams();
			checkClientStreams();

			// make request from client to remote server
			String request = requestType + " " + uri + " HTTP/1.0";
			remoteOutputStream.write(request.getBytes());
			remoteOutputStream.write(endOfLine.getBytes());
			System.out.println(request);

			// send hostname
			String command = "host: "+ header.get("host");
			remoteOutputStream.write(command.getBytes());
			remoteOutputStream.write(endOfLine.getBytes());
			System.out.println(command);

			// send rest of the headers
			for( String key : header.keySet()) {
				if(!key.equals("host")){
					command = key + ": "+ header.get(key);
					remoteOutputStream.write(command.getBytes());
					remoteOutputStream.write(endOfLine.getBytes());
					System.out.println(command);
				}
			}

			remoteOutputStream.write(endOfLine.getBytes());
			remoteOutputStream.flush();

			// send client request data if its a POST request
			if(requestType.startsWith("POST")) {

				int contentLength = Integer.parseInt(header.get("content-length"));
				for (int i = 0; i < contentLength; i++)
				{
					remoteOutputStream.write(clientInputStream.read());
				}
			}

			// complete remote server request
			remoteOutputStream.write(endOfLine.getBytes());
			remoteOutputStream.flush();
		}
		catch (UnknownHostException e) { return; } 
		catch (SocketException e){ return; } 
		catch (IOException e) { return;} 
	}

	/**
	 * Sending buffered remote server response back to client with minor header processing
	 * 
	 */
	@SuppressWarnings("deprecation")
	private void remoteToClient() {

		try {

			// If socket is closed, return
			if(remoteSocket == null) return;

			String line;
			DataInputStream remoteOutHeader = new DataInputStream(remoteSocket.getInputStream());

			// get remote response header
			while((line = remoteOutHeader.readLine()) != null) {

				// check for end of header blank line
				if(line.trim().length() == 0) break;

				// check for proxy-connection: keep-alive
				if(line.toLowerCase().startsWith("proxy")) continue;
				if(line.contains("keep-alive")) continue;

				// write remote response to client
				System.out.println(line);
				clientOutputStream.write(line.getBytes());
				clientOutputStream.write(endOfLine.getBytes());
			}

			// complete remote header response
			clientOutputStream.write(endOfLine.getBytes());
			clientOutputStream.flush();

			// get remote response body
			remoteInputStream = remoteSocket.getInputStream();
			byte[] buffer = new byte[1024];

			// buffer remote response then write it back to client
			for(int i; (i = remoteInputStream.read(buffer)) != -1;) 
			{
				clientOutputStream.write(buffer, 0, i);
				clientOutputStream.flush();
			}
		} 
		catch (UnknownHostException e) { return; } 
		catch (SocketException e){ return; } 
		catch (IOException e) { return;} 
	}

	/**
	 * Helper function to strip out unwanted request header from client
	 * 
	 */
	private void stripUnwantedHeaders() {

		if(header.containsKey("user-agent")) header.remove("user-agent");
		if(header.containsKey("referer")) header.remove("referer");
		if(header.containsKey("proxy-connection")) header.remove("proxy-connection");
		if(header.containsKey("connection") && header.get("connection").equalsIgnoreCase("keep-alive")) {
			header.remove("connection");
		}
	}

	/**
	 * Helper function to check for client input and output stream, reconnect if closed
	 * 
	 */
	private void checkClientStreams() {

		try {
			if(clientSocket.isOutputShutdown())	clientOutputStream = clientSocket.getOutputStream();
			if(clientSocket.isInputShutdown())	clientInputStream = new DataInputStream(clientSocket.getInputStream());
		}
		catch (UnknownHostException e) { return; } 
		catch (SocketException e){ return; } 
		catch (IOException e) { return;} 
	}

	/**
	 * Helper function to check for remote input and output stream, reconnect if closed
	 * 
	 */
	private void checkRemoteStreams() {

		try {
			if(remoteSocket.isOutputShutdown())	remoteOutputStream = remoteSocket.getOutputStream();
			if(remoteSocket.isInputShutdown())	remoteInputStream = new DataInputStream(remoteSocket.getInputStream());
		} 
		catch (UnknownHostException e) { return; } 
		catch (SocketException e){ return; } 
		catch (IOException e) { return;} 
	}

	/**
	 * Helper function to parse URI from full URL
	 * 
	 */
	private void getUri() {

		if(header.containsKey("host")) 
		{
			int temp = url.indexOf(header.get("host"));
			temp += header.get("host").length();

			if(temp < 0) { 
				// prevent index out of bound, use entire url instead
				uri = url;
			} else {
				// get uri from part of the url
				uri = url.substring(temp);	
			}
		}
	}

}
