# FP_SISOP19_A14

### -- Daemon Thread --

##### Buatlah program C yang menyerupai crontab menggunakan daemon dan thread. Ada sebuah file crontab.data untuk menyimpan config dari crontab. Setiap ada perubahan file tersebut maka secara otomatis program menjalankan config yang sesuai dengan perubahan tersebut tanpa perlu diberhentikan. Config hanya sebatas * dan 0-9 (tidak perlu /,- dan yang lainnya)

Source Code :
```sh
#include <sys/types.h>
#include <sys/stat.h>
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <errno.h>
#include <unistd.h>
#include <syslog.h>
#include <string.h>
#include <time.h>
#include <pthread.h>
#include <sys/wait.h>

int read_line(char str[], FILE *fi);
void *c_ron(void *ye);
void do_this(char ln[], struct dis *strc);

char *from = "/home/Penunggu/fpsisop/crontab.data";

struct dis
{
    int c_time[5];
    int fin;
    char where[100];
    char pathis[100];
    double tdif;
    char a_time[100];
    char wat_to_do[100];
};

int main()
{
    pid_t pid, sid;

    pid = fork();

    if (pid < 0)
    {
        exit(EXIT_FAILURE);
    }

    if (pid > 0)
    {
        exit(EXIT_SUCCESS);
    }

    umask(0);

    sid = setsid();

    if (sid < 0)
    {
        exit(EXIT_FAILURE);
    }

    if ((chdir("/")) < 0)
    {
        exit(EXIT_FAILURE);
    }

    // close(STDIN_FILENO);
    // close(STDOUT_FILENO);
    // close(STDERR_FILENO);

    FILE *ct;

    struct dis *rc;
    rc = malloc(10 * sizeof(struct dis));

    pthread_t threads[10];

    struct stat fi_st;

    time_t lt, mt;
    time(&lt);

    int lc = -1;

    char ln[100];
    int hitung;

    while (1)
    {
        stat(from, &fi_st);
        mt = fi_st.st_mtime;
        if (mt != lt)
        {
            ct = fopen(from, "r");

            fseek(ct, 0, SEEK_SET);

            lt = mt;

            hitung = 0;

            printf("akses = %s\n", lt);
            printf("modif = %s\n", mt);

            while (read_line(ln, ct) != -1)
            {
                hitung++;

                struct dis *pk = rc + hitung;

                strcpy(pk->where, ln);

                do_this(ln, pk);

                if (lc < hitung)
                {
                    pthread_create(&threads[hitung], NULL, c_ron, (void *)pk);
                }
            }
            int z;

            for(z = hitung; z <= lc; z++)
            {
                struct dis *tmp = rc + z;

                tmp->fin = 1;
            }

            lc = hitung;

            fclose(ct);

            ct = NULL;
        }

        else
        {
          printf("hmm\n");
        }
        sleep(1);
    }

    exit(EXIT_SUCCESS);
}

void do_this(char ln[], struct dis *strc)
{
    //ngambil * * * * *
    int i = 0, t = 0, hitung;
    char tmp[10];
    while (i < 5)
    {
        hitung = 0;
        while (ln[t] != ' ')
        {
            tmp[hitung] = ln[t];
            hitung++;
            t++;
        }
        if (tmp[0] != '*')
            strc->c_time[i] = atoi(tmp);
        else
            strc->c_time[i] = -1;
        t++;
        i++;
    }

    strcpy(strc->a_time, &ln[t]);

    strcpy(strc->pathis, &ln[t]);

    if (strstr(strc->pathis, " ") != NULL)
    {
        strcpy(strc->wat_to_do, strstr(strc->pathis, " ") + 1);
        strc->pathis[strlen(strc->pathis) - strlen(strc->wat_to_do) - 1] = '\0';
    }
    else
    {
        strcpy(strc->wat_to_do, "\0");
    }

    strc->fin = 0;
}

int read_line(char str[], FILE *fi)
{
    char y;
    int k = 0;

    memset(str, '\0', 100 * sizeof(char));

    while (fscanf(fi, "%c", &y) != EOF)
    {
        if (y == '\n')
        {
            return 0;
        }
        str[k] = y;

        k++;
    }

    printf("%s\n", str);

    if (strcmp(str, "\0") == 0)
    {
      return -1;
    }

    return 0;
}

void *c_ron(void *ye)
{
    struct dis *ar = (struct dis *)ye;
    struct stat fi_st;
    struct tm *curt;


    time_t rr, chk;

    while (ar->fin == 0)
    {
        time(&rr);

        curt = localtime(&rr);

        if (ar->c_time[0] < 0 || ar->c_time[0] == curt->tm_min)
        {
            if (ar->c_time[1] < 0 || ar->c_time[1] == curt->tm_hour)
            {
                if (ar->c_time[2] < 0 || ar->c_time[2] == curt->tm_mday)
                {
                    if (ar->c_time[3] < 0 || ar->c_time[3] == curt->tm_mon + 1)
                    {
                        if (ar->c_time[4] < 0 || ar->c_time[4] == curt->tm_wday)
                        {
                            // printf("%s\n", ar->where);

                            int child = fork();

                            int statt;

                            if(child == 0)
                            {
                                if(ar->wat_to_do[0] != '\0')
                                {
                                    char *ag[] = {ar->wat_to_do, ar->pathis, NULL};

                                    execv(ar->wat_to_do, ag);
                                }

                                else
                                {
                                    char *ag[] = {ar->pathis, NULL};
                                    execv(ar->pathis, ag);
                                }
                            }

                            else
                            {
                                while(wait(&statt) > 0);
                            }

                            time(&chk);

                            while (difftime(chk, rr) <= 60.0)
                            {
                                sleep(1);
                                time(&chk);
                            }
                        }
                    }
                }
            }
        }
    }
}

```
