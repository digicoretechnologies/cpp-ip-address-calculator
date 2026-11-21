#include <cstdint>
#include <iostream>
#include <string>

using namespace std;

// Validate and convert IPv4 address
bool parseIPv4(const string& address, uint32_t& ip) {
    size_t start = 0;
    ip = 0;

    for (int i = 0; i < 4; ++i) {
        size_t end = address.find('.', start);
        string part;

        if (i < 3) {
            if (end == string::npos)
                return false;

            part = address.substr(start, end - start);
        } else {
            if (end != string::npos)
                return false;

            part = address.substr(start);
        }

        if (part.empty())
            return false;

        for (char c : part) {
            if (c < '0' || c > '9')
                return false;
        }

        if (part.size() > 3)
            return false;

        unsigned int octet = 0;

        for (char c : part) {
            octet = octet * 10 +
                    static_cast<unsigned int>(c - '0');
        }

        if (octet > 255)
            return false;

        ip = (ip << 8) | octet;

        start = (i < 3) ? end + 1 : address.size();
    }

    return true;
}

// Convert integer into IPv4 format
string formatIPv4(uint32_t value) {
    return to_string((value >> 24) & 255u) + "." +
           to_string((value >> 16) & 255u) + "." +
           to_string((value >> 8) & 255u) + "." +
           to_string(value & 255u);
}

int main() {

    string ipAddress;
    int prefix;
    uint32_t ip = 0;

    cout << "===== IPv4 SUBNET CALCULATOR =====\n\n";

    cout << "Enter IPv4 address: ";

    if (!(cin >> ipAddress)) {
        cerr << "Error: Could not read IP address.\n";
        return 1;
    }

    if (!parseIPv4(ipAddress, ip)) {
        cerr << "Error: Invalid IPv4 address.\n";
        return 1;
    }

    cout << "Enter CIDR prefix (0-32): /";

    if (!(cin >> prefix) || prefix < 0 || prefix > 32) {
        cerr << "Error: Prefix must be between 0 and 32.\n";
        return 1;
    }

    // Calculate subnet mask
    uint32_t mask = 0;

    if (prefix == 0) {
        mask = 0;
    } else {
        mask = 0xFFFFFFFFu << (32 - prefix);
    }

    // Calculate network and last address
    uint32_t network = ip & mask;
    uint32_t lastAddress = network | ~mask;

    // Calculate host information
    int hostBits = 32 - prefix;

    uint64_t totalAddresses = 1ULL << hostBits;

    uint64_t usableHosts;

    if (prefix <= 30) {
        usableHosts = totalAddresses - 2;
    }
    else if (prefix == 31) {
        usableHosts = 2;
    }
    else {
        usableHosts = 1;
    }

    // Display network report
    cout << "\n========== NETWORK REPORT ==========\n";

    cout << "IP Address          : "
         << formatIPv4(ip) << endl;

    cout << "CIDR Prefix         : /"
         << prefix << endl;

    cout << "Subnet Mask         : "
         << formatIPv4(mask) << endl;

    cout << "Network Address     : "
         << formatIPv4(network) << endl;

    if (prefix <= 30) {
        cout << "Broadcast Address   : "
             << formatIPv4(lastAddress) << endl;
    }
    else {
        cout << "Broadcast Address   : Not applicable for /"
             << prefix << endl;

        cout << "Last Address        : "
             << formatIPv4(lastAddress) << endl;
    }

    cout << "Host Bits           : "
         << hostBits << endl;

    cout << "Total Addresses     : "
         << totalAddresses << endl;

    cout << "Usable Host Addresses: "
         << usableHosts << endl;

    if (prefix == 31) {
        cout << "Note: /31 is intended for point-to-point links.\n";
    }
    else if (prefix == 32) {
        cout << "Note: /32 identifies one individual address.\n";
    }

    cout << "====================================\n";

    return 0;
}
