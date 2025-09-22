import java.io.*;
import java.net.*;
import java.util.*;

public class ChatServer {
    // List to hold all client handlers (threads)
    private static final List<ClientHandler> clients = new ArrayList<>();

    public static void main(String[] args) {
        int port = 12345;  // Port for the server to listen on
        System.out.println("Chat Server started...");

        try (ServerSocket serverSocket = new ServerSocket(port)) {
            while (true) {
                // Accept incoming client connections
                Socket clientSocket = serverSocket.accept();
                System.out.println("New client connected: " + clientSocket.getInetAddress());

                // Create a new thread for the client
                ClientHandler clientHandler = new ClientHandler(clientSocket);
                clients.add(clientHandler);
                new Thread(clientHandler).start();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    // Broadcast message to all clients
    public static void broadcastMessage(String message) {
        for (ClientHandler client : clients) {
            client.sendMessage(message);
        }
    }

    // Remove client from the list of clients
    public static void removeClient(ClientHandler client) {
        clients.remove(client);
    }

    // Client handler class (handles each client's communication)
    static class ClientHandler implements Runnable {
        private final Socket socket;
        private final PrintWriter out;
        private final BufferedReader in;

        public ClientHandler(Socket socket) throws IOException {
            this.socket = socket;
            this.out = new PrintWriter(socket.getOutputStream(), true);
            this.in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
        }

        @Override
        public void run() {
            String clientName = socket.getInetAddress().toString();
            try {
                out.println("Welcome to the chat, " + clientName);
                String message;
                while ((message = in.readLine()) != null) {
                    if (message.equalsIgnoreCase("exit")) {
                        break;
                    }
                    System.out.println(clientName + ": " + message);
                    // Broadcast the message to all clients
                    broadcastMessage(clientName + ": " + message);
                }
            } catch (IOException e) {
                e.printStackTrace();
            } finally {
                // Remove client and close resources
                removeClient(this);
                try {
                    socket.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

        // Send message to the client
        public void sendMessage(String message) {
            out.println(message);
        }
    }
}
