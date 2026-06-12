# DecodeLabs-Internship-P2
DecodeLabs-Internship Cybersecurity batchc Project 2
// ============================================================
// DecodeLabs Cybersecurity Internship | Batch 2026
// Project 2: Basic Encryption & Decryption — Caesar Cipher
// Implements the IPO Model: Plaintext → Algorithm+Key → Cipher
// Formula: E(x) = (x + n) % 26   |   D(x) = (x - n + 26) % 26
// ============================================================

#include <iostream>
#include <string>
#include <cctype>
#include <limits>
using namespace std;

// ── core cipher engine ────────────────────────────────────────

/*
 * Shifts a single alphabetic character by `shift` positions.
 * Preserves case; non-alpha characters pass through unchanged.
 * Uses modular arithmetic to handle wrap-around (e.g. Z+3 = C).
 */
char shiftChar(char c, int shift) {
    if (isalpha(static_cast<unsigned char>(c))) {
        char base = isupper(static_cast<unsigned char>(c)) ? 'A' : 'a';
        // Normalise shift to [0,25] so negative shifts work too
        shift = ((shift % 26) + 26) % 26;
        return static_cast<char>((c - base + shift) % 26 + base);
    }
    return c;   // spaces, digits, punctuation unchanged
}

/** Encrypt plaintext with Caesar cipher using key n. */
string encrypt(const string& plaintext, int key) {
    string ciphertext;
    ciphertext.reserve(plaintext.size());
    for (char c : plaintext)
        ciphertext += shiftChar(c, key);
    return ciphertext;
}

/** Decrypt ciphertext by applying the reverse shift. */
string decrypt(const string& ciphertext, int key) {
    return encrypt(ciphertext, -key);   // Symmetric: same key, opposite sign
}

// ── display helpers ───────────────────────────────────────────
const string RESET = "\033[0m";
const string BOLD = "\033[1m";
const string CYAN = "\033[1;36m";
const string GREEN = "\033[1;32m";
const string YELLOW = "\033[1;33m";
const string RED = "\033[1;31m";

void printHeader() {
    cout << CYAN
        << "╔══════════════════════════════════════════════╗\n"
        << "║  DecodeLabs — Caesar Cipher Encrypt/Decrypt  ║\n"
        << "║              Batch 2026 | Project 2          ║\n"
        << "╚══════════════════════════════════════════════╝\n"
        << RESET << "\n";
}

void showAlgorithmSteps(char example, int key) {
    char base = std::isupper(static_cast<unsigned char>(example)) ? 'A' : 'a';
    int  xPos = example - base;
    int  shifted = (xPos + key) % 26;
    char result = static_cast<char>(shifted + base);

    cout << YELLOW
        << "  Algorithm walkthrough for '" << example << "' with key " << key << ":\n"
        << "    1. ASCII → position  : '" << example << "' = " << xPos << "\n"
        << "    2. Add key           : " << xPos << " + " << key << " = " << (xPos + key) << "\n"
        << "    3. Modulo 26         : " << (xPos + key) << " % 26 = " << shifted << "\n"
        << "    4. Back to char      : " << shifted << " → '" << result << "'\n"
        << RESET;
}

void printSeparator() {
    cout << "──────────────────────────────────────────────\n";
}

// ── main menu ─────────────────────────────────────────────────
int getValidKey() {
    int key;
    while (true) {
        cout << BOLD << "Enter shift key (1-25): " << RESET;
        if (cin >> key && key >= 1 && key <= 25) {
           cin.ignore(numeric_limits<streamsize>::max(), '\n');
            return key;
        }
        cin.clear();
        cin.ignore(numeric_limits<streamsize>::max(), '\n');
        cout << RED << "  [!] Invalid key. Please enter a number between 1 and 25.\n" << RESET;
    }
}

int main() {
    printHeader();

    char menuChoice;
    do {
        cout << BOLD
            << "Choose an option:\n"
            << "  [1] Encrypt a message\n"
            << "  [2] Decrypt a message\n"
            << "  [3] Encrypt then verify by decrypting\n"
            << "  [q] Quit\n"
            << "Selection: " << RESET;

        cin >> menuChoice;
        cin.ignore(std::numeric_limits<streamsize>::max(), '\n');
        cout << "\n";

        if (menuChoice == '1') {
            // ── ENCRYPT ──────────────────────────────────────
            string plaintext;
            cout << BOLD << "Enter plaintext: " << RESET;
            getline(cin, plaintext);

            int key = getValidKey();

            string ciphertext = encrypt(plaintext, key);

            printSeparator();
           cout << GREEN << "  Plaintext  : " << RESET << plaintext << "\n";
           cout << YELLOW << "  Shift key  : " << RESET << key << "\n";
            cout << CYAN << "  Ciphertext : " << RESET << ciphertext << "\n";
            printSeparator();

            // Show the algorithm walkthrough for the first alpha char
            for (char c : plaintext) {
                if (isalpha(static_cast<unsigned char>(c))) {
                    showAlgorithmSteps(c, key);
                    break;
                }
            }

        }
        else if (menuChoice == '2') {
            // ── DECRYPT ──────────────────────────────────────
            string ciphertext;
            cout << BOLD << "Enter ciphertext: " << RESET;
            getline(cin, ciphertext);

            int key = getValidKey();

           string plaintext = decrypt(ciphertext, key);

            printSeparator();
            cout << CYAN << "  Ciphertext : " << RESET << ciphertext << "\n";
            cout << YELLOW << "  Shift key  : " << RESET << key << "\n";
            cout << GREEN << "  Plaintext  : " << RESET << plaintext << "\n";
            printSeparator();

        }
        else if (menuChoice == '3') {
            // ── ENCRYPT + VERIFY ─────────────────────────────
            string plaintext;
            cout << BOLD << "Enter plaintext: " << RESET;
            getline(cin, plaintext);

            int key = getValidKey();

            string decrypted = decrypt(ciphertext, key);
            string ciphertext = encrypt(plaintext, key);
            bool        verified = (decrypted == plaintext);

            printSeparator();
            cout << GREEN << "  Original   : " << RESET << plaintext << "\n";
            cout << YELLOW << "  Key        : " << RESET << key << "\n";
            cout << CYAN << "  Encrypted  : " << RESET << ciphertext << "\n";
            cout << GREEN << "  Decrypted  : " << RESET << decrypted << "\n";
            cout << (verified ? GREEN : RED)
                << "  Verified   : " << (verified ? "✓ Match — symmetric key works!" : "✗ Mismatch!")
                << RESET << "\n";
            printSeparator();

            // Algorithm walkthrough
            for (char c : plaintext) {
                if (isalpha(static_cast<unsigned char>(c))) {
                    showAlgorithmSteps(c, key);
                    break;
                }
            }

        }
        else if (menuChoice != 'q' && menuChoice != 'Q') {
            cout << RED << "  [!] Invalid option. Choose 1, 2, 3, or q.\n" << RESET;
        }

        cout << "\n";

    } while (menuChoice != 'q' && menuChoice != 'Q');

  cout << CYAN << "Data confidentiality mastered! — DecodeLabs 2026\n" << RESET;
    return 0;
}
