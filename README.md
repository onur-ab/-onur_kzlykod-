#include <stdio.h>
#include <stdlib.h>
#include <time.h>

#define N 9

int araDeger(int *satir, int deger) {
    int i;

    for (i = 0; i < N; i++) {
        if (satir[i] == deger) {
            return 0;
        }
    }

    return 1;
}

void matrisiYazdir(int **matris) {
    int i, j;

    printf("\n     1   2   3   4   5   6   7   8   9");
    for (i = 0; i < N; i++) {
        printf("\n  --------------------------------------\n%d |  ", i + 1);
        for (j = 0; j < N; j++) {
            printf("%d   ", matris[i][j]);
        }
    }
    printf("\n  --------------------------------------\n");
}

int araDegerKarede(int **sudoku, int deger, int *kare) {
    int i, j;

    for (i = kare[0]; i <= kare[1]; i++) {
        for (j = kare[2]; j <= kare[3]; j++) {
            if (deger == sudoku[i][j]) {
                return 0;
            }
        }
    }

    return 1;
}

int **matrisAyir() {
    int i;
    int **matris;

    // kullaniciSudoku ve sudoku için calloc ile yer ayırıp sıfırlıyoruz
    matris = (int **)calloc(N, sizeof(int *));
    if (matris == NULL) {
        printf("HATA: Bellek ayırma başarısız!");
        exit(0);
    }
    for (i = 0; i < N; i++) {
        matris[i] = (int *)calloc(N, sizeof(int));
        if (matris[i] == NULL) {
            printf("HATA: Bellek ayırma başarısız!");
            exit(0);
        }
    }

    return matris;
}

void menuyuYazdir(int *op) {
    do {
        printf("\n        OYUN MENÜSÜ");
        printf("\n        ------------");
        printf("\n    1 - Sayı Yaz");
        printf("\n    2 - Sayı Sil");
        printf("\n    3 - Yeni Oyun");
        printf("\n    4 - Nasıl Oynanır?");
        printf("\n    5 - Oynamak İstemiyorum - Çıkış\n");

        printf("\nYapmak istediğiniz işlem > ");
        scanf("%d", op);
    } while (*op < 1 || *op > 5);
}

void nasilOynanir() {
    printf("\n\n SUDOKU NASIL OYNANIR\n");
    printf(" ---------------------");
    printf("\n # 9 satır ve sütundan oluşan karede, 1-9 arasındaki sayılar");
    printf("\naynı satırda ve aynı sütunda tekrar etmemelidir.\n");
    printf("\n # Ayrıca büyük kare içerisinde oluşan 3x3'lük 9 karecik");
    printf("\niçerisinde de 1-9 arasındaki sayılar birer kere kullanılmalıdır.\n");
    printf("\nDevam etmek için bir tuşa basınız > ");
    getchar();
    getchar();
}

void koordinatlariAl(int *satir, int *sutun) {
    do {
        printf("\nKoordinatlar 1-9 arasında olmalıdır (dahil)\n");
        printf("Satır > ");
        scanf("%d", satir);
        printf("Sütun > ");
        scanf("%d", sutun);
    } while ((*satir > 9 || *satir < 1) || (*sutun < 1 || *sutun > 9));

    // Matrislerimiz 0'dan başladığı için birer azaltalım
    (*satir)--;
    (*sutun)--;
}

int karsilastir(int **sudoku, int **kullaniciSudoku) {
    int i, j;

    for (i = 0; i < N; i++) {
        for (j = 0; j < N; j++) {
            if (sudoku[i][j] != kullaniciSudoku[i][j]) {
                return 0; // yanlış çözmüş
            }
        }
    }
    return 1; // doğru çözmüş
}

int main() {
    int **sudoku;
    int **kullaniciSudoku;
    int aralik[9][2];
    int kareler[9][4] = { {0, 2, 0, 2},
                          {3, 5, 0, 2},
                          {6, 8, 0, 2},
                          {0, 2, 3, 5},
                          {3, 5, 3, 5},
                          {6, 8, 3, 5},
                          {0, 2, 6, 8},
                          {3, 5, 6, 8},
                          {6, 8, 6, 8},
                        };
    int i, j, k, z;
    int indeks;
    int kareIndeks;
    int deneme = 0;
    int islem;
    int satir, sutun;
    int girilenSayi = 20; // 81 karenin 20'sini önceden girip kullanıcıya sunuyoruz.

    // Rastgele değerlerin her çalıştırıldığında farklı olması için
    srand(time(NULL));

    // 9x9 luk kareler için yer ayır.
    sudoku = matrisAyir();
    kullaniciSudoku = matrisAyir();

    // Sudoku'da 1-9 arası sayılar olacak. Onları dizinin 1. satırına yerleştirelim.
    // 2. satırda ise sayının o anki sudoku sütununda kullanılıp kullanılmadığı değerini tutacak
    // kullanıldı = 1, kullanılmamış = 0
    for (i = 0; i < 9; i++) {
        aralik[i][0] = i + 1;
        aralik[i][1] = 0;
    }

    // Sudokunun oluşturulması.
    j = 0;
    i = 0;
    while (j != N) { // sütun sayacı

        if (i == 100) {
            // break ile çıkmış
            j--; // break ile çıktığımız için en alta j artı. Sileceğimiz sütun bir önceki olmalı.
            for (z = 0; z < N; z++) {
                sudoku[z][j] = 0;
            }
        }

        // aralik 2. satırını sıfırla
        for (k = 0; k < N; k++) {
            aralik[k][1] = 0;
        }

        deneme = 0;
        i = 0;

        while (i != N) {
            // Eğer 100 kerede sayı bulunmamışsa ters giden bir şeyler var. O sütuna sayıları baştan yerleştireceğiz.
            if (deneme > 100) {
                i = 100;
                break;
            }

            indeks = rand() % 9; // for aralik index [0-8]

            // Gelen sayı daha önce o sütunda kullanılmış mı?
            if (aralik[indeks][1] == 1) {
                continue;
            }

            // Sayının yerleştirilemediği başarısız olduğumuz durum sayısı
            deneme++;

            // Gelen sayı o anki satırda kullanılmış mı?
            if (!(araDeger(sudoku[i], aralik[indeks][0]))) {
                continue;
            }

            // Satır(i) ve sütun(j) durumuna göre o an 9 küçük kareden oluşan karelerden hangisindeyiz?
            if (i < 3) {
                if (j < 3) {
                    kareIndeks = 0;
                } else if (j < 6) {
                    kareIndeks = 3;
                } else {
                    kareIndeks = 6;
                }
            } else if (i < 6) {
                if (j < 3) {
                    kareIndeks = 1;
                } else if (j < 6) {
                    kareIndeks = 4;
                } else {
                    kareIndeks = 7;
                }
            } else {
                if (j < 3) {
                    kareIndeks = 2;
                } else if (j < 6) {
                    kareIndeks = 5;
                } else {
                    kareIndeks = 8;
                }
            }

            // Seçilen sayı o anki 9 lu hücrenin içinde var mı?
            if (!(araDegerKarede(sudoku, aralik[indeks][0], kareler[kareIndeks]))) {
                continue;
            }

            sudoku[i][j] = aralik[indeks][0];
            deneme = 0;
            aralik[indeks][1] = 1;
            i++;
        } // i while'ın sonu
        j++;
    }     // j while'ın sonu

    // Kullanıcıya göstereceğimiz sudoku'da bazı elemanları önceden yerleştirmemiz lazım
    // 15 tanesini yerleştirelim.
    i = 0;
    while (i != 20) {
        satir = rand() % 9;
        sutun = rand() % 9;

        if (kullaniciSudoku[satir][sutun] == 0) {
            kullaniciSudoku[satir][sutun] = sudoku[satir][sutun];
            i++;
        }
    }

    printf("\n Sudoku oluşturuldu!\n");

    do {
        matrisiYazdir(kullaniciSudoku);
        printf("\n");
        menuyuYazdir(&islem);

        switch (islem) {
        case 1: // Sayı gir
            koordinatlariAl(&satir, &sutun);
            if (kullaniciSudoku[satir][sutun] != 0) {
                printf("\nHATA : İstediğimiz bölgede zaten değer var!\n");
            } else {
                int deger;
                do {
                    printf("\n[%d, %d] bölgesine yazacağınız değer [1-9]> ", satir + 1, sutun + 1);
                    scanf("%d", &deger);
                } while (deger > 9 || deger < 0);
                kullaniciSudoku[satir][sutun] = deger;
                girilenSayi++;
                printf("\nİstediğiniz bölgeye istediğiniz değer yazıldı.\n");
            }
            break;
        case 2: // Sayı sil
            koordinatlariAl(&satir, &sutun);
            if (kullaniciSudoku[satir][sutun] == 0) {
                printf("\nHATA : İstediğimiz bölgede zaten değer yok!\n");
            } else {
                kullaniciSudoku[satir][sutun] = 0;
                girilenSayi--;
                printf("\nİstediğiniz bölgedeki değer silindi!\n");
            }
            break;
        case 3: // Yeni oyun
            printf("\nYeni oyun için programı kapatıp tekrar açın :) \n");
            return 0;
            break;
        case 4: // Nasıl oynanır
            nasilOynanir();
            break;
        case 5: // çıkış
            printf("Çıkılıyor....");
            break;
        // default'a gerek yok. Gerekli kontrol menuyuYazdir fonksiyonu yapıldı.
        }
    } while (islem != 5 || girilenSayi != 81);

    if (girilenSayi == 81) {
        // Kullanıcı sudoku'yu tamamlamış bakalım doğru mu?
        if (karsilastir(sudoku, kullaniciSudoku)) {
            printf("\nTEBRİKLER! Sudoku'yu başarıyla çözdünüz!\n");
        } else {
            printf("\nMaalesef! Çözümünüzde yanlışlıklar var!\n");
        }
    }

    free(sudoku);
    free(kullaniciSudoku);
    return 0;
}
