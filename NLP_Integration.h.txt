#include <string>

std::string processNLP(const std::string& user_input) {
    // Simulate NLP Response
    if (user_input == "hello") {
        return "Hi there! How can I assist you?";
    } else if (user_input == "bye") {
        return "Goodbye!";
    } else {
        return "I'm not sure, but I'll try my best to help!";
    }
}
