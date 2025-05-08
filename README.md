# Sisop-3-2025-IT11

### MEMBER
1. Muhammad Ardiansyah Tri Wibowo - 5027241091
2. Erlinda Annisa Zahra Kusuma - 5027241108
3. Fika Arka Nuriyah - 5027241071


### REPORTING 

### SOAL 1
### SOAL 2
RushGo ingin memberikan layanan ekspedisi dengan 2 pilihan ada reguler dan express. lalu membuat delivery management system untuk RushGo.
Sistem ini memiliki 2 bagian utama:
- delivery_agent.c sebagai agent pengantar express
- dispatcher.c sebagai pengiriman reguler dan monitoring oleh user

a. Mengunduh file delivery_order.csv lalu membaca seluruh data dari CSV dan menyimpan ke shared memory.

Mengunduh file

      if (!file_exists(CSV_FILENAME)) {
        printf("File %s tidak ditemukan. Mengunduh...\n", CSV_FILENAME);

        pid_t pid = fork();
        if (pid == 0) {
            // Child process
            char* args[] = {"wget", "-O", CSV_FILENAME, DOWNLOAD_LINK, NULL};
            execvp("wget", args);
            // Jika execvp gagal
            perror("execvp gagal");
            exit(EXIT_FAILURE);
        } else if (pid > 0) {
            // Parent process
            int status;
            waitpid(pid, &status, 0);
            if (WIFEXITED(status) && WEXITSTATUS(status) != 0) {
                fprintf(stderr, "Gagal mengunduh file CSV.\n");
                return 1;
            }
        } else {
            // Fork gagal
            perror("fork gagal");
            return 1;
        }

        printf("Berhasil mengunduh %s\n", CSV_FILENAME);
    }

    FILE* file = fopen(CSV_FILENAME, "r");
    if (!file) {
        perror("Gagal membuka file CSV");
        return 1;
    }

Membaca File CSV

      FILE* file = fopen(CSV_FILENAME, "r");
    if (!file) {
        perror("Gagal membuka file CSV");
        return 1;
    }

Menyimpan ke Shared Memory

      int shmid = shmget(SHM_KEY, sizeof(SharedMemory), IPC_CREAT | 0666);
    if (shmid == -1) {
        perror("shmget gagal");
        fclose(file);
        return 1;
    }

    SharedMemory* shm = (SharedMemory*)shmat(shmid, NULL, 0);
    if (shm == (void*)-1) {
        perror("shmat gagal");
        fclose(file);
        return 1;
    }


b. Pengiriman bertipe express
RushGo disini memakai 3 agent ada agent A,B dan C. Agent akan secara otomatis mencari order bertipe express yang belum dikirim. Mengambil dan mengirimkannya tanpa user. lalu akan di catat dalam log.

            
        if (strcmp(shm->orders[shm->order_count].type, "Express") == 0) {
            char agent = 'A' + (rand() % 3); // A, B, atau C
            strcpy(shm->orders[shm->order_count].status, "Delivered");
            fprintf(log_file, "%s [AGENT %c] Express package delivered to %s in %s\n",
                    timestamp,
                    agent,
                    shm->orders[shm->order_count].name,
                    shm->orders[shm->order_count].address);
        }

        shm->order_count++;
    }

lalu dicatat dalam log 

            char timestamp[32];
            get_timestamp(timestamp, sizeof(timestamp));


c. Pengiriman bertipe reguler
Disini pengirimannya dilakukan dengan agent <user>. User disini dapat mengirim dengan menggunakan perintah dispatcher.

Pengiriman dengan reguler

            if (strcmp(argv[1], "-deliver") == 0 && argc == 3) {
        const char* recipient = argv[2];
        int found = 0;

        for (int i = 0; i < shm->order_count; i++) {
            if (strcmp(shm->orders[i].name, recipient) == 0 &&
                strcmp(shm->orders[i].type, "Reguler") == 0 &&
                strcmp(shm->orders[i].status, "Pending") == 0) {

                strcpy(shm->orders[i].status, "Delivered");
                log_delivery("Reguler", shm->orders[i].name, shm->orders[i].address);
                printf("Reguler order for '%s' marked as Delivered.\n", recipient);
                found = 1;
                break;
            }
        }

d. Mengecek Status pesanan

            void check_status(SharedMemory *shm, const char* name) {
    for (int i = 0; i < shm->order_count; i++) {
        if (strcmp(shm->orders[i].name, name) == 0) {
            if (strcmp(shm->orders[i].status, "Delivered") == 0) {
                FILE *log_file = fopen("delivery.log", "r");
                if (!log_file) {
                    perror("Gagal membuka delivery.log");
                    return;
                }

                char line[256];
                char agent_name[100] = "UNKNOWN";
                while (fgets(line, sizeof(line), log_file)) {
                    if (strstr(line, name)) {
                        char *start = strstr(line, "[AGENT ");
                        if (start) {
                            start += 7; // lewati "[AGENT "
                            char *end = strchr(start, ']');
                            if (end) {
                                *end = '\0';
                                strncpy(agent_name, start, sizeof(agent_name));
                                break;
                            }
                        }
                    }
                }
                fclose(log_file);

                printf("Status for %s: Delivered by Agent %s\n", name, agent_name);

e. Melihat daftar semua pesanan
Bisa menjalankan perintah list untuk melihat semua order nama disertai status

            void list_orders(SharedMemory *shm) {
    printf("Listing all orders:\n");
    for (int i = 0; i < shm->order_count; i++) {
        printf("Name: %s, Status: %s\n", shm->orders[i].name, shm->orders[i].status);
          }
      } 
            
            

### SOAL 3

Pada tahap ini terdapat 8 point yang perlu dibuat dalam menyelesaikan permasalahan ini.

a. dungeon.c dan player.c berjalan dengan komunikasi RPC, dan server dapat menjalankan banyak client dalam 1 server.

#### dungeon.C
```
// Library yang perlu digunakan
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <string.h>
#include <pthread.h>
#include <stdbool.h>
....

int main(){
    ...
    // sesuai yang ada di modul
    pthread_mutex_init(&client_lock, NULL); // Inisialisasi mutex

    if(( servernya = socket(AF_INET, SOCK_STREAM, 0)) == 0){
        perror("socket failed");
        exit(EXIT_FAILURE);
    }

    alamat.sin_family = AF_INET;
    alamat.sin_addr.s_addr = INADDR_ANY;
    alamat.sin_port = htons(PORT);

    if(bind(servernya, (struct sockaddr *)&alamat, sizeof(alamat)) < 0){
        perror("bind failed");
        exit(EXIT_FAILURE);
    }

    if(listen(servernya, 3) < 0){
        perror("listen failed");
        exit(EXIT_FAILURE);
    }
    ...

    while(1){
        printf("Menunggu koneksi...\n");
        int* new_socket = (int*)malloc(sizeof(int));
        if ((*new_socket = accept(servernya, (struct sockaddr*)&alamat, (socklen_t*)&addrlen)) < 0) {
            perror("accept failed");
            free(new_socket);
            continue; // lanjut ke iterasi berikutnya
        }
        printf("Koneksi diterima\n");

        // Buat thread baru untuk menangani client
        pthread_t thread_id;
        if (pthread_create(&thread_id, NULL, handle_client, new_socket) != 0) {
            perror("pthread_create failed");
            close(*new_socket);
            free(new_socket);
            continue; // lanjut ke iterasi berikutnya
        }

        // Detach thread agar sumber daya dibebaskan setelah selesai
        pthread_detach(thread_id);
    }
}

```

#### dungeon.c (dalam handle banyak client)
```
void handle_client(...){
...
while (1) {
    int pilihan_dari_client;
    int bytes_read = read(socketnya, &pilihan_dari_client, sizeof(pilihan_dari_client));
    if (bytes_read < 0) {
        perror("read failed");
        break; // Keluar dari perulangan jika terjadi error
    }
    if (bytes_read == 0) {
        printf("Client dengan port %d terputus.\n", client_port);
        break; // Keluar dari perulangan jika client menutup koneksi
    }
...
}
```

### player.c 
```
int main(){
    ....

    if ((sock = socket(AF_INET, SOCK_STREAM, 0)) < 0) {
        perror("Socket creation failed");
        exit(EXIT_FAILURE);
    }

    serv_addr.sin_family = AF_INET;
    serv_addr.sin_port = htons(PORT);

    if (inet_pton(AF_INET, IP, &serv_addr.sin_addr) <= 0) {
        perror("Invalid address/ Address not supported");
        exit(EXIT_FAILURE);
    }

    if (connect(sock, (struct sockaddr *)&serv_addr, sizeof(serv_addr)) < 0) {
        // Jika koneksi gagal, kita bisa menampilkan pesan error
        loading(20);
        fprintf(stderr, "\033[1;31m");
        perror("Connection failed");
        fprintf(stderr, "\033[0m");

        exit(EXIT_FAILURE);
    }
    ....
}
```

b. Menampilkan main menu, dan listener hubungan dungeon.c dan player.c

#### player.C
```
void logo(){
printf("\033[H\033[J");
printf("██████╗░██╗░░░██╗███╗░░██╗░██████╗░███████╗░█████╗░███╗░░██╗\n");
printf("██╔══██╗██║░░░██║████╗░██║██╔════╝░██╔════╝██╔══██╗████╗░██║\n");
printf("██║░░██║██║░░░██║██╔██╗██║██║░░██╗░█████╗░░██║░░██║██╔██╗██║\n");
printf("██║░░██║██║░░░██║██║╚████║██║░░╚██╗██╔══╝░░██║░░██║██║╚████║\n");
printf("██████╔╝╚██████╔╝██║░╚███║╚██████╔╝███████╗╚█████╔╝██║░╚███║\n");
printf("╚═════╝░░╚═════╝░╚═╝░░╚══╝░╚═════╝░╚══════╝░╚════╝░╚═╝░░╚══╝\n\n");
}

void menu(){
    printf("\033[H\033[J");
    logo();
    printf("1. Show Player Stats\n");
    printf("2. Inventory\n");
    printf("3. Shop (Buy Weapons)\n");
    printf("4. Battle\n");
    printf("5. Exit Game\n\n");
    printf(RED "   Choose an option: " RESET);

    int pilihanku;
    scanf("%d", &pilihanku);
    switch (pilihanku) {
        ....
    }

}    
```

c. Menampilkan status check 
### dungeon.c (proses background)
```
....
if (send(socketnya, &player[nomor_player], sizeof(player[nomor_player]), 0) <= 0) {
    perror("send failed (client stats)");
    break; // Keluar dari perulangan jika terjadi error
}
printf("Stats dikirim ke client dengan port %d\n", client_port);
....
```
### SOAL 4
