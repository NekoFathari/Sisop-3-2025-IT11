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


### SOAL 3
### SOAL 4
