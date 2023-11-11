# Server-Client

import java.io.*;
import java.net.*;

public class Server {
    public static void main(String[] args) throws IOException {
        ServerSocket serverSocket = new ServerSocket(1234); // порт сервера

        Socket clientSocket = serverSocket.accept();
        System.out.println("Соединение установлено с клиентом");

        // Потоки для чтения/записи данных
        BufferedReader in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
        PrintWriter out = new PrintWriter(clientSocket.getOutputStream(), true);

        String message;

        // Запись истории чата в файл
        FileWriter fileWriter = new FileWriter("chat_history.txt", true);
        PrintWriter logWriter = new PrintWriter(fileWriter, true);

        // Чтение истории чата из файла и отправка клиенту при подключении
        BufferedReader historyReader = new BufferedReader(new FileReader("chat_history.txt"));
        String line;
        while ((line = historyReader.readLine()) != null) {
            out.println(line);
        }
        historyReader.close();

        // Основной цикл чтения/отправки сообщений
        while ((message = in.readLine()) != null) {
            System.out.println("Получено сообщение от клиента: " + message);
            out.println(message); // пересылка сообщения клиенту

            // Запись сообщения в файл
            logWriter.println(message);
        }

        // Закрытие соединения
        in.close();
        out.close();
        clientSocket.close();
        serverSocket.close();
        logWriter.close();
    }
}




import java.io.*;
import java.net.*;
import javax.swing.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.awt.event.KeyEvent;
import java.awt.event.KeyListener;

public class Client {
    public static void main(String[] args) throws IOException {
        Socket socket = new Socket("localhost", 1234); // адрес и порт сервера

        // Создание графического интерфейса
        JFrame frame = new JFrame("Chat Client");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setSize(400, 400);
        JTextArea historyArea = new JTextArea();
        JTextArea messageArea = new JTextArea();
        JButton sendButton = new JButton("Отправить");
        frame.getContentPane().add(historyArea, "Center");
        frame.getContentPane().add(messageArea, "South");
        frame.getContentPane().add(sendButton, "East");
        frame.setVisible(true);

        // Потоки для чтения/записи данных
        BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
        PrintWriter out = new PrintWriter(socket.getOutputStream(), true);

        // Чтение истории чата из файла
        File file = new File("chat_history.txt");
        if (file.exists()) {
            BufferedReader historyReader = new BufferedReader(new FileReader("chat_history.txt"));
            String line;
            while ((line = historyReader.readLine()) != null) {
                historyArea.append(line + "\n");
            }
            historyReader.close();
        }

        // Отправка сообщения по кнопке
        sendButton.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                String message = messageArea.getText();
                out.println(message);
                messageArea.setText("");
            }
        });

        // Отправка сообщения по нажатию Enter
        messageArea.addKeyListener(new KeyListener() {
            public void keyTyped(KeyEvent e) {
            }

            public void keyPressed(KeyEvent e) {
                if (e.getKeyCode() == KeyEvent.VK_ENTER) {
                    String message = messageArea.getText();
                    out.println(message);
                    messageArea.setText("");
                }
            }

            public void keyReleased(KeyEvent e) {
            }
        });


// Основной цикл чтения/отображения сообщений
        String message;
        while ((message = in.readLine()) != null) {
            System.out.println("Получено сообщение от сервера: " + message);
            historyArea.append(message + "\n");
        }

// Закрытие соединения
        in.close();
        out.close();
        socket.close();
    }
}




