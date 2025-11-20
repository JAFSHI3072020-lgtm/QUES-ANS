#include <iostream>
#include <vector>
#include <string>
#include <cctype>
#include <cstdlib>
#include <ctime>
#include <csignal>
#include <unistd.h>
#include <termios.h>

using namespace std;

struct Question {
    string text;
    vector<string> options;
    char correct_option;
    int timeout;
    int prize_money;
};

volatile int timeout_happened = 0;

const string COLOR_END = "\033[0m";
const string RED = "\033[1;31m";
const string GREEN = "\033[1;32m";
const string YELLOW = "\033[1;33m";
const string BLUE = "\033[1;34m";
const string PINK = "\033[1;35m";
const string AQUA = "\033[1;36m";

struct termios old_props;

void reset_terminal_attributes() {
    tcsetattr(STDIN_FILENO, TCSANOW, &old_props);
}

void set_terminal_attributes() {
    tcgetattr(STDIN_FILENO, &old_props);
    atexit(reset_terminal_attributes);
    struct termios new_props = old_props;
    new_props.c_lflag &= ~(ECHO | ICANON);
    tcsetattr(STDIN_FILENO, TCSANOW, &new_props);
}

void timeout_handler(int) {
    timeout_happened = 1;
    cout << RED << "\n\nTime out!!!!!  Press Any Key..." << COLOR_END << endl;
    fflush(stdout);
}

void print_formatted_question(const Question& q) {
    cout << "\n\n" << YELLOW << q.text << COLOR_END << endl;
    for (int i = 0; i < 4; i++) {
        if (!q.options[i].empty()) {
            cout << AQUA << char('A' + i) << ". " << q.options[i] << COLOR_END << endl;
        }
    }
    cout << YELLOW << "Hurry!! You have only " << q.timeout << " Seconds to answer.." << COLOR_END << endl;
    cout << GREEN << "Enter your answer (A, B, C, or D) or L for lifeline: " << COLOR_END;
}

int use_lifeline(Question& q, int* lifeline) {
    cout << "\n\n" << PINK << "Available Lifelines:" << COLOR_END;
    if (lifeline[0]) cout << "\n" << PINK << "1. Fifty-Fifty (50/50)" << COLOR_END;
    if (lifeline[1]) cout << "\n" << PINK << "2. Skip the Question" << COLOR_END;
    cout << "\n" << PINK << "Choose a lifeline or 0 to return: " << COLOR_END;

    char ch;
    cin >> ch;

    switch (ch) {
        case '1':
            if (lifeline[0]) {
                lifeline[0] = 0;
                int removed = 0;
                while (removed < 2) {
                    int num = rand() % 4;
                    if ((num + 'A') != q.correct_option && !q.options[num].empty()) {
                        q.options[num] = "";
                        removed++;
                    }
                }
                return 1;
            }
            break;
        case '2':
            if (lifeline[1]) {
                lifeline[1] = 0;
                return 2;
            }
            break;
        default:
            cout << "\n" << PINK << "Returning to the Question." << COLOR_END << endl;
            break;
    }
    return 0;
}

void play_game(vector<Question>& questions) {
    int money_won = 0;
    int lifeline[] = {1, 1};

    signal(SIGALRM, timeout_handler);

    for (size_t i = 0; i < questions.size(); i++) {
        print_formatted_question(questions[i]);
        alarm(questions[i].timeout);

        char ch;
        cin >> ch;
        alarm(0);
        ch = toupper(ch);

        if (timeout_happened == 1) break;

        if (ch == 'L') {
            int value = use_lifeline(questions[i], lifeline);
            if (value != 2) i--;
            continue;
        }

        if (ch == questions[i].correct_option) {
            cout << GREEN << "\nCorrect!" << COLOR_END << endl;
            money_won = questions[i].prize_money;
            cout << BLUE << "\nYou have won: Rs " << money_won << COLOR_END << endl;
        } else {
            cout << RED << "\nWrong! Correct answer is " << questions[i].correct_option << "." << COLOR_END << endl;
            break;
        }
    }
    cout << "\n\n" << BLUE << "Game Over! Your total winnings are: Rs " << money_won << COLOR_END << endl;
}

int main() {
    srand(time(NULL));
    set_terminal_attributes();

    cout << "\t\t" << PINK << "Chalo Kehlte hain KAUN BANEGA CROREPATI !!!" << COLOR_END << endl;

    vector<Question> questions = {
        {"What is the capital city of Bangladesh?", {"Chittagong","Dhaka","Sylhet","Khulna"}, 'B', 10, 1000},
        {"What is the currency of Bangladesh?", {"Rupee","Ringgit","Taka","Riyal"}, 'C', 10, 2000},
        {"In which year did Bangladesh gain its independence?", {"1965","1971","1952","1947"}, 'B', 10, 3000},
        {"Which animal is the national animal of Bangladesh?", {"Asian Elephant","One-horned Rhinoceros","Bengal Fox","Royal Bengal Tiger"}, 'D', 10, 5000},
        {"Who is regarded as the Father of the Nation of Bangladesh?", {"Ziaur Rahman","Sheikh Mujibur Rahman","Huseyn Shaheed Suhrawardy","Muhammad Ali Jinnah"}, 'B', 10, 10000},
        {"What is the state language of Bangladesh?", {"Urdu","Hindi","Bengali (Bangla)","English"}, 'C', 10, 20000},
        {"Which city in Bangladesh is famous for having the world's longest natural sea beach?", {"Patuakhali","Kuakata","Cox's Bazar","Saint Martin's Island"}, 'C', 15, 40000},
        {"Which famous forest, shared with India, is the largest mangrove forest in the world?", {"Lawachara Forest","Rema-Kalenga Wildlife Sanctuary","Sundarbans","Satchari National Park"}, 'C', 15, 80000},
        {"Who was the architect of the Jatiya Sangsad Bhaban (National Parliament House) of Bangladesh?", {"Fazlur Rahman Khan","Louis Kahn","Muzharul Islam","Charles Correa"}, 'B', 15, 160000},
        {"What is the national flower of Bangladesh?", {"Rose","Water Lily (Shapla)","Marigold","Lotus"}, 'B', 15, 320000},
        {"What is the national bird of Bangladesh?", {"Kingfisher","Dove","Magpie Robin (Doyel)","Sparrow"}, 'C', 30, 640000},
        {"What is the primary crop of Bangladesh, essential to its agriculture and diet?", {"Wheat","Rice","Maize","Potato"}, 'B', 30, 1250000},
        {"The Language Movement, a pivotal event in Bangladesh's history, took place in which year?", {"1952","1969","1970","1971"}, 'A', 30, 2500000},
        {"What is the name of the highest peak in Bangladesh?", {"Keokradong","Tazing Dong (Bijoy)","Saka Haphong","Chimbuk"}, 'B', 30, 5000000},
        {"What is the nickname of the Bangladesh national cricket team?", {"The Tigers","The Lions","The Royal Bengals","The Green Brigade"}, 'A', 30, 10000000}
    };

    play_game(questions);
    return 0;
}
