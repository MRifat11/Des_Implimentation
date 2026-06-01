#include <iostream>
#include <string>
#include <iomanip>
using namespace std;

// Initial Permutation
int IP[] = {
58,50,42,34,26,18,10,2,
60,52,44,36,28,20,12,4,
62,54,46,38,30,22,14,6,
64,56,48,40,32,24,16,8,
57,49,41,33,25,17,9,1,
59,51,43,35,27,19,11,3,
61,53,45,37,29,21,13,5,
63,55,47,39,31,23,15,7
};

// Final Permutation
int FP[] = {
40,8,48,16,56,24,64,32,
39,7,47,15,55,23,63,31,
38,6,46,14,54,22,62,30,
37,5,45,13,53,21,61,29,
36,4,44,12,52,20,60,28,
35,3,43,11,51,19,59,27,
34,2,42,10,50,18,58,26,
33,1,41,9,49,17,57,25
};

// Expansion (32 -> 48)
int E[] = {
32,1,2,3,4,5,
4,5,6,7,8,9,
8,9,10,11,12,13,
12,13,14,15,16,17,
16,17,18,19,20,21,
20,21,22,23,24,25,
24,25,26,27,28,29,
28,29,30,31,32,1
};

// P permutation (32 -> 32)
int P[] = {
16,7,20,21,
29,12,28,17,
1,15,23,26,
5,18,31,10,
2,8,24,14,
32,27,3,9,
19,13,30,6,
22,11,4,25
};

// PC-1 (key 64 -> 56)
int PC1[] = {
57,49,41,33,25,17,9,
1,58,50,42,34,26,18,
10,2,59,51,43,35,27,
19,11,3,60,52,44,36,
63,55,47,39,31,23,15,
7,62,54,46,38,30,22,
14,6,61,53,45,37,29,
21,13,5,28,20,12,4
};

// PC-2 (key 56 -> 48)
int PC2[] = {
14,17,11,24,1,5,
3,28,15,6,21,10,
23,19,12,4,26,8,
16,7,27,20,13,2,
41,52,31,37,47,55,
30,40,51,45,33,48,
44,49,39,56,34,53,
46,42,50,36,29,32
};

// Left shifts per round
int SHIFTS[] = {1,1,2,2,2,2,2,2,1,2,2,2,2,2,2,1};

// S-boxes (8 x 4 x 16)
int SBOX[8][4][16] = {
    { // S1
        {14,4,13,1,2,15,11,8,3,10,6,12,5,9,0,7},
        {0,15,7,4,14,2,13,1,10,6,12,11,9,5,3,8},
        {4,1,14,8,13,6,2,11,15,12,9,7,3,10,5,0},
        {15,12,8,2,4,9,1,7,5,11,3,14,10,0,6,13}
    },
    { // S2
        {15,1,8,14,6,11,3,4,9,7,2,13,12,0,5,10},
        {3,13,4,7,15,2,8,14,12,0,1,10,6,9,11,5},
        {0,14,7,11,10,4,13,1,5,8,12,6,9,3,2,15},
        {13,8,10,1,3,15,4,2,11,6,7,12,0,5,14,9}
    },
    { // S3
        {10,0,9,14,6,3,15,5,1,13,12,7,11,4,2,8},
        {13,7,0,9,3,4,6,10,2,8,5,14,12,11,15,1},
        {13,6,4,9,8,15,3,0,11,1,2,12,5,10,14,7},
        {1,10,13,0,6,9,8,7,4,15,14,3,11,5,2,12}
    },
    { // S4
        {7,13,14,3,0,6,9,10,1,2,8,5,11,12,4,15},
        {13,8,11,5,6,15,0,3,4,7,2,12,1,10,14,9},
        {10,6,9,0,12,11,7,13,15,1,3,14,5,2,8,4},
        {3,15,0,6,10,1,13,8,9,4,5,11,12,7,2,14}
    },
    { // S5
        {2,12,4,1,7,10,11,6,8,5,3,15,13,0,14,9},
        {14,11,2,12,4,7,13,1,5,0,15,10,3,9,8,6},
        {4,2,1,11,10,13,7,8,15,9,12,5,6,3,0,14},
        {11,8,12,7,1,14,2,13,6,15,0,9,10,4,5,3}
    },
    { // S6
        {12,1,10,15,9,2,6,8,0,13,3,4,14,7,5,11},
        {10,15,4,2,7,12,9,5,6,1,13,14,0,11,3,8},
        {9,14,15,5,2,8,12,3,7,0,4,10,1,13,11,6},
        {4,3,2,12,9,5,15,10,11,14,1,7,6,0,8,13}
    },
    { // S7
        {4,11,2,14,15,0,8,13,3,12,9,7,5,10,6,1},
        {13,0,11,7,4,9,1,10,14,3,5,12,2,15,8,6},
        {1,4,11,13,12,3,7,14,10,15,6,8,0,5,9,2},
        {6,11,13,8,1,4,10,7,9,5,0,15,14,2,3,12}
    },
    { // S8
        {13,2,8,4,6,15,11,1,10,9,3,14,5,0,12,7},
        {1,15,13,8,10,3,7,4,12,5,6,11,0,14,9,2},
        {7,11,4,1,9,12,14,2,0,6,10,13,15,3,5,8},
        {2,1,14,7,4,10,8,13,15,12,9,0,3,5,6,11}
    }
};

// Helper functions

string permute(const string &in, int *table, int n) {
    string out = "";
    for (int i = 0; i < n; i++) out += in[table[i] - 1];
    return out;
}

string leftShift(const string &s, int n) {
    string res = s;
    for (int i = 0; i < n; i++) res = res.substr(1) + res[0];
    return res;
}

string xorBits(const string &a, const string &b) {
    string res = "";
    for (size_t i = 0; i < a.size(); i++)
        res += (a[i] == b[i]) ? '0' : '1';
    return res;
}

string sboxSub(const string &in48) {
    string out32 = "";
    for (int b = 0; b < 8; b++) {
        string block = in48.substr(b*6, 6);
        int row = (block[0]-'0')*2 + (block[5]-'0');
        int col = (block[1]-'0')*8 + (block[2]-'0')*4 +
                  (block[3]-'0')*2 + (block[4]-'0');
        int val = SBOX[b][row][col];
        string bits = "";
        for (int i = 3; i >= 0; i--)
            bits += ((val>>i)&1) ? '1' : '0';
        out32 += bits;
    }
    return out32;
}

string feistel(const string &R, const string &K) {
    string ER = permute(R, E, 48);     // 32 -> 48
    string x  = xorBits(ER, K);        // XOR with subkey
    string sb = sboxSub(x);            // 48 -> 32
    string p  = permute(sb, P, 32);    // permutation P
    return p;
}

// char <-> bits
string charToBin(char c) {
    string s = "";
    for (int i = 7; i >= 0; i--)
        s += ((c>>i)&1) ? '1' : '0';
    return s;
}

char binToChar(const string &s8) {
    int val = 0;
    for (int i = 0; i < 8; i++)
        val = val*2 + (s8[i]-'0');
    return (char)val;
}

string textToBin(const string &txt) {
    string res = "";
    for (size_t i = 0; i < txt.size(); i++)
        res += charToBin(txt[i]);
    return res;
}

string binToText(const string &bin) {
    string res = "";
    for (size_t i = 0; i < bin.size(); i += 8)
        res += binToChar(bin.substr(i, 8));
    return res;
}

// Key schedule (16 subkeys)

void generateSubkeys(const string &key64, string subkeys[16]) {
    string key56 = permute(key64, PC1, 56);  // 64 -> 56
    string C = key56.substr(0,28);
    string D = key56.substr(28,28);

    for (int round = 0; round < 16; round++) {
        C = leftShift(C, SHIFTS[round]);
        D = leftShift(D, SHIFTS[round]);
        string CD = C + D;
        subkeys[round] = permute(CD, PC2, 48); // 56 -> 48
    }
}

// 1 block DES (encrypt/decrypt) 
string desBlock(string block64, string subkeys[16], bool encrypt) {
    string ip = permute(block64, IP, 64);
    string L  = ip.substr(0,32);
    string R  = ip.substr(32,32);

    for (int round = 0; round < 16; round++) {
        string K = encrypt ? subkeys[round] : subkeys[15-round];
        string f = feistel(R, K);
        string newR = xorBits(L, f);
        L = R;
        R = newR;
    }

    string preoutput = R + L;              // swap
    string fp = permute(preoutput, FP, 64); // final permutation
    return fp;
}

// This is main function

int main() {
    string plain, key;
    cout << "Enter 8-char plaintext: ";
    getline(cin, plain);
    cout << "Enter 8-char key: ";
    getline(cin, key);

    if (plain.size() != 8 || key.size() != 8) {
        cout << "Please enter exactly 8 characters for plaintext and key." << endl;
        return 0;
    }

    string plainBin = textToBin(plain); // 64 bits
    string keyBin   = textToBin(key);   // 64 bits

    string subkeys[16];
    generateSubkeys(keyBin, subkeys);

    // Encrypt
    string cipherBin  = desBlock(plainBin, subkeys, true);
    string cipherText = binToText(cipherBin);

    cout << "\nEncrypted (hex bytes): ";
    for (size_t i = 0; i < cipherText.size(); i++) {
        unsigned char ch = cipherText[i];
        cout << hex << uppercase << setw(2) << setfill('0') << (int)ch << ' ';
    }
    cout << dec << endl; // back to decimal

    // Decrypt
    string decryptedBin  = desBlock(cipherBin, subkeys, false);
    string decryptedText = binToText(decryptedBin);

    cout << "\nDecrypted text: " << decryptedText << endl;

    return 0;
}
