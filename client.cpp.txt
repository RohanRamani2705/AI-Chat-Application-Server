#include <iostream>
#include <thread>
#include <vector>
#include <mutex>
#include <map>
#include <netinet/in.h>
#include <unistd.h>
#include "NLP_Integration.h" // NLP API integration

#define PORT 8080
#define BUFFER_SIZE 1024

std::mutex clients_mutex;
std::map<int, std::string> clients;

void handle_client(int client_socket) {
    char buffer[BUFFER_SIZE];
    while (true) {
        int bytes_received = recv(client_socket, buffer, BUFFER_SIZE, 0);
        if (bytes_received <= 0) {
            std::cout << "Client disconnected\n";
            close(client_socket);
            break;
        }

        buffer[bytes_received] = '\0';
        std::string message(buffer);

        // NLP Integration
        std::string response = processNLP(message);

        std::cout << "Client: " << message << "\nServer: " << response << std::endl;
        send(client_socket, response.c_str(), response.length(), 0);
    }
}

int main() {
    int server_fd, client_socket;
    struct sockaddr_in address;
    int opt = 1;
    int addrlen = sizeof(address);

    server_fd = socket(AF_INET, SOCK_STREAM, 0);
    setsockopt(server_fd, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));

    address.sin_family = AF_INET;
    address.sin_addr.s_addr = INADDR_ANY;
    address.sin_port = htons(PORT);

    bind(server_fd, (struct sockaddr*)&address, sizeof(address));
    listen(server_fd, 10);

    std::cout << "Server started on port " << PORT << std::endl;

    while (true) {
        client_socket = accept(server_fd, (struct sockaddr*)&address, (socklen_t*)&addrlen);
        std::cout << "New connection established\n";
        std::thread(handle_client, client_socket).detach();
    }

    close(server_fd);
    return 0;
}
