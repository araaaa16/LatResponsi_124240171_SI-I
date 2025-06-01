#include <iostream>
#include <string>
#include <iomanip>
using namespace std;

struct Riwayat {
    string tanggal, keterangan;
    Riwayat* next;
};

struct Pelaku {
    string nama;
    string posisi;
    double nominal;
    Riwayat* riwayat;
    Pelaku* left;
    Pelaku* right;

    Pelaku(string n = "", string p = "", double nom = 0.0)
        : nama(n), posisi(p), nominal(nom), riwayat(nullptr), left(nullptr), right(nullptr) {}
};

struct Aksi {
    string tipe;
    Pelaku* data;

    Aksi(string t = "", Pelaku* d = nullptr) : tipe(t), data(d) {}
};

class AuditSystem {
private:
    Pelaku* root;
    Aksi lastAction;

    void tambahRiwayat(Pelaku* p, string tgl, string ket) {
        Riwayat* baru = new Riwayat{tgl, ket, nullptr};
        if (!p->riwayat) p->riwayat = baru;
        else {
            Riwayat* temp = p->riwayat;
            while (temp->next) temp = temp->next;
            temp->next = baru;
        }
    }

    Pelaku* insertBST(Pelaku* node, Pelaku* baru) {
        if (!node) return baru;
        if (baru->nama < node->nama)
            node->left = insertBST(node->left, baru);
        else if (baru->nama > node->nama)
            node->right = insertBST(node->right, baru);
        return node;
    }

    Pelaku* searchBST(Pelaku* node, const string& nama) {
        if (!node || node->nama == nama) return node;
        if (nama < node->nama) return searchBST(node->left, nama);
        return searchBST(node->right, nama);
    }

    Pelaku* minNode(Pelaku* node) {
        while (node && node->left) node = node->left;
        return node;
    }

    Pelaku* deleteBST(Pelaku* node, const string& nama, Pelaku*& deleted) {
        if (!node) return nullptr;
        if (nama < node->nama)
            node->left = deleteBST(node->left, nama, deleted);
        else if (nama > node->nama)
            node->right = deleteBST(node->right, nama, deleted);
        else {
            deleted = new Pelaku(node->nama, node->posisi, node->nominal);
            deleted->riwayat = node->riwayat;

            if (!node->left) {
                Pelaku* temp = node->right;
                delete node;
                return temp;
            } else if (!node->right) {
                Pelaku* temp = node->left;
                delete node;
                return temp;
            }
            Pelaku* temp = minNode(node->right);
            node->nama = temp->nama;
            node->posisi = temp->posisi;
            node->nominal = temp->nominal;
            node->riwayat = temp->riwayat;
            node->right = deleteBST(node->right, temp->nama, deleted); // dummy delete
        }
        return node;
    }

    void inorder(Pelaku* node) {
        if (!node) return;
        inorder(node->left);
        cout << "Nama: " << node->nama
             << " | Posisi: " << node->posisi
             << " | Nominal: Rp " << fixed << setprecision(2) << node->nominal << endl;
        Riwayat* r = node->riwayat;
        while (r) {
            cout << "    Riwayat: [" << r->tanggal << "] " << r->keterangan << endl;
            r = r->next;
        }
        inorder(node->right);
    }

public:
    AuditSystem() : root(nullptr), lastAction() {}

    void tambahPelaku() {
        string nama, posisi;
        double nominal;

        cout << "\n=== TAMBAH DATA TERDUGA PELAKU ===" << endl;
        cout << "Nama Pelaku: ";
        cin.ignore();
        getline(cin, nama);

        if (searchBST(root, nama)) {
            cout << "Data dengan nama '" << nama << "' sudah ada!" << endl;
            return;
        }

        cout << "Posisi: ";
        getline(cin, posisi);
        cout << "Nominal Dana (Rp): ";
        cin >> nominal;

        Pelaku* baru = new Pelaku(nama, posisi, nominal);
        tambahRiwayat(baru, "2025-06-01", "Ditambahkan ke sistem.");

        root = insertBST(root, baru);
        lastAction = Aksi("tambah", baru);

        cout << "Data berhasil ditambahkan!" << endl;
    }

    void tampilkanDaftar() {
        cout << "\n=== DAFTAR TERDUGA PELAKU (A-Z) ===" << endl;
        if (!root) {
            cout << "Tidak ada data pelaku." << endl;
        } else {
            inorder(root);
        }
    }

    void hapusPelaku() {
        string nama;
        cout << "\n=== HAPUS DATA PELAKU ===" << endl;
        cout << "Masukkan nama pelaku yang akan dihapus: ";
        cin.ignore();
        getline(cin, nama);

        Pelaku* temp = nullptr;
        root = deleteBST(root, nama, temp);
        if (temp) {
            lastAction = Aksi("hapus", temp);
            cout << "Data pelaku '" << nama << "' berhasil dihapus!" << endl;
        } else {
            cout << "Data pelaku '" << nama << "' tidak ditemukan!" << endl;
        }
    }

    void undoAksi() {
        if (lastAction.tipe == "") {
            cout << "\nTidak ada aksi yang dapat dibatalkan." << endl;
            return;
        }

        cout << "\n=== UNDO AKSI TERAKHIR ===" << endl;
        if (lastAction.tipe == "tambah") {
            Pelaku* dummy = nullptr;
            root = deleteBST(root, lastAction.data->nama, dummy);
            cout << "Penambahan data '" << lastAction.data->nama << "' telah dibatalkan." << endl;
        } else if (lastAction.tipe == "hapus") {
            root = insertBST(root, lastAction.data);
            cout << "Penghapusan data '" << lastAction.data->nama << "' telah dibatalkan." << endl;
        }
        lastAction = Aksi(); // reset
    }

    void tampilkanMenu() {
        cout << "\n========================================" << endl;
        cout << "  SISTEM AUDIT INTERNAL MBG" << endl;
        cout << "========================================" << endl;
        cout << "1. Tambah Data Terduga Pelaku" << endl;
        cout << "2. Tampilkan Daftar Pelaku" << endl;
        cout << "3. Hapus Data Pelaku" << endl;
        cout << "4. Undo Aksi Terakhir" << endl;
        cout << "5. Keluar" << endl;
        cout << "========================================" << endl;
        cout << "Pilihan Anda: ";
    }
};

int main() {
    AuditSystem sistem;
    int pilihan;

    cout << "Selamat datang di Sistem Audit Internal" << endl;
    cout << "Yayasan Media Berkat Gemilang" << endl;

    do {
        sistem.tampilkanMenu();
        cin >> pilihan;

        switch (pilihan) {
            case 1: sistem.tambahPelaku(); break;
            case 2: sistem.tampilkanDaftar(); break;
            case 3: sistem.hapusPelaku(); break;
            case 4: sistem.undoAksi(); break;
            case 5:
                cout << "\nTerima kasih telah menggunakan sistem!" << endl;
                break;
            default:
                cout << "\nPilihan tidak valid. Silakan coba lagi." << endl;
        }

        if (pilihan != 5) {
            cout << "\nKlik Enter untuk melanjutkan...";
            cin.ignore();
            cin.get();
        }

    } while (pilihan != 5);

    return 0;
}
