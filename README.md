# P1runningback

#include <string.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdio.h>
#include <dirent.h>
#include <limits.h>
#include <math.h>
#include <pwd.h>
#include <stdbool.h>
#include <stdlib.h>
#include <errno.h>

#include <ctype.h>


#include "debug.h"
#include "procfs.h"

void strip_extra_spaces(char* str){
    int i, x;
    for(i=x=0; str[i]; ++i){
        if(!isspace(str[i]) || (i > 0 && !isspace(str[i-1]))){
            str[x++] = str[i];
        }
    }
    str[x] = '\0';

}

int pfs_hostname(char *proc_dir, char *hostname_buf, size_t buf_sz)
{
    char extra[] = "/sys/kernel/hostname";
    char extra2[100];
    int read_sz;
    strcpy(extra2, proc_dir);
    strcat(extra2, extra);
     //getting hostname
    int fd;
    fd = open(extra2, O_RDONLY, O_DIRECTORY);
    //LOG("fd: %d\n", fd);    
    if (fd <= 2){
    perror("There was a problem");
    exit(0);
    }    

    read_sz = read(fd, hostname_buf, buf_sz);
    //LOG("read_sz: %d, buf: %2\n", read_sz, hostname_buf);   
    
    if (read_sz <= 0) {
        //printf("error with read\n");
    return -1;
    } else {
        int i =0;
       
        strip_extra_spaces(hostname_buf);
        while(hostname_buf[i] !='\0'){
           if (hostname_buf[i] == '\n'){
              hostname_buf[i] = '\0';
            }
           i++;
        }
            close(fd);
    }
    return 0;
}


int isFile(const char* filename)
{
    DIR* directory = opendir(filename);
    if(directory != NULL) {
        closedir(directory);
        return 0;
    }
    if(errno == ENOTDIR) {
        return 1;
    }
    return -1;
}

char *next_token(char **str_ptr, const char *delim)
{
    if (*str_ptr == NULL) {
        return NULL;
    }

    size_t tok_start = strspn(*str_ptr, delim);
    size_t tok_end = strcspn(*str_ptr + tok_start, delim);
    /* Zero length token. We must be finished. */
    if (tok_end  <= 0) {
        *str_ptr = NULL;
        return NULL;
    }


/* Take note of the start of the current token. We'll return it later. */
    char *current_ptr = *str_ptr + tok_start;
    /* Shift pointer forward (to the end of the current token) */
    *str_ptr += tok_start + tok_end;
    if (**str_ptr == '\0') {
         *str_ptr = NULL;
    } else {
         **str_ptr = '\0';
         (*str_ptr)++;
    }
    return current_ptr;
}


int pfs_kernel_version(char *proc_dir, char *version_buf, size_t buf_sz)
{
    char v[buf_sz];
    int read_sz;
    char vdir[] = "/version";
    strcpy(v, proc_dir);
    strcat(v, vdir);

    char buf[buf_sz];
    int fd;
    
    fd = open(v, O_RDONLY);
    if (fd <= 2){
        perror("errors on open");                     
    exit(0);
    }
    
    read_sz = read(fd, buf, buf_sz);
    char *next_tok =buf;
    //strcpy(next_tok, buf);
    char *curr_tok;
    bool nextTok = false;

    char version[] = "version";

/* Tokenize. Note that ' ,?!' will all be removed. */

    while ((curr_tok = next_token(&next_tok, " ")) != NULL) {
        if (nextTok == true && ( strcmp(curr_tok, version) != 0) ) {
        nextTok = false ;
        break;
    } else{
        if (strcmp(curr_tok, version) == 0){
            nextTok = true; 
        }
    }
     }
    strcpy(version_buf, curr_tok);

    close(fd);
//    version_buf = curr_toprintf("Kernel Version: %s\n", curr_tok);
    return -1;
}

int pfs_cpu_model(char *proc_dir, char *model_buf, size_t buf_sz)
{
    char buf[buf_sz];
    //ssize_t read_sz;
    int fd;
    char cpu[10000];
    char cpudir[] = "/cpuinfo";
    strcpy(cpu, proc_dir);
    strcat(cpu, cpudir);
    
    fd = open(cpu, O_RDONLY);
    
    if (fd <=2){
        perror("open");                     
        exit(0);    
        //return EXIT_FAILURE;
    }
   
    read(fd, buf, buf_sz);
    int tokens = 0;
    char *next_tok = buf;
    char *curr_tok;
    char *model = "model name";

    bool nextTok = false;
    /* Tokenize. Note that ' ,?!' will all be removed. */
    while ((curr_tok = next_token(&next_tok, "\t:\n")) != NULL) {
        if (nextTok == true) {
    //      printf("CPU MODEL: %s\n", curr_tok);
            strcpy(model_buf, curr_tok);
            nextTok = false;
            break;
        } else if (strcmp(curr_tok, model) == 0){
            nextTok = true;
        }
    tokens++;
    }

    close(fd);
    return -1;
}

int pfs_cpu_units(char *proc_dir)
{
    //char buf[buf_sz];
        ssize_t read_sz;
    char tewbuf[10000]; 
       
    int fd;
    char punits[10000];
    char pdir[] = "/stat";
    strcpy(punits, proc_dir);
    strcat(punits, pdir);
    char *curr_tok;
    fd = open(punits, O_RDONLY);
    if (fd <= 2){
        perror("open");
        exit(0);    
    }
    int ret = 0;
    int len =0;

    ///find the total bytes each line takes up and make array with it
    //and then loop throug it accessing the 
    
    char* cref  = "cpu";
    read_sz = read(fd, tewbuf, 10000 );
    char *next_tok = tewbuf;
    len =0;

    while ((curr_tok= next_token(&next_tok, "\n")) != NULL) {
        len = strspn(curr_tok, cref);
        if (len == 3) {
            ret++;  
        }
    }

    ret = ret - 1;
    //printf("Processing Units: %d\n", ret);
    
    close(fd);
    return ret;
    
    //return 0;
}

double pfs_uptime(char *proc_dir)
{
    char up[1000];
    char updir[] = "/uptime";
        strcpy(up, proc_dir);
    strcat(up, updir);
    char buf[128];
    
    int read_sz;
    int fd;
    char *next_tok;
    char *curr_tok;
    fd = open(up, O_RDONLY);
    //check for fake friend files(i.e. invalid files)
    if (fd <= 2){
        perror("open");
        exit(0);
    }


    read_sz = read(fd, buf, 128);
    next_tok = buf;
    curr_tok = next_token(&next_tok, " ");
    
     double curr = 0.0;
     curr = atof(curr_tok);
     return curr;
    //return 0.0;
}

int pfs_format_uptime(double time, char *uptime_buf)
{
    int yr=0, dy=0, hr =0, m =0, seco=0;
    char yrstr[50], dystr[50], hrstr[50];

    int ref = (int)time;
    if (time >= (365*24*60*60)){
        yr = (int)time / (365*24*60*60);
        //snprintf(yrstr, 50, "%f", yr);
            //printf("%d years, ", yr);
        ref = (int)time % (365*24*60*60);
    } 
    if (ref >= (24*60*60)){
        dy = ref / (24*60*60);
        //snprintf(dystr, 50, "%f", dy);
        //printf("%d days, ", dy);
        ref = ref % (24*60*60); 
    }
    if (ref >= (60*60)) {
        hr = ref / (60*60);
        //snprintf(hrstr, 50, "%f", hr);
        //printf("%d hours, ", hr);
        ref = ref % (60*60);
    }
    //char output[50];
    
    //snprintf(output, 50, "%f", num);
    m = ref/60;
    seco = ref - (m * 60);
        snprintf(uptime_buf, sizeof(uptime_buf), "%d years, %d days, %d hours, %d minutes, %d seconds", yr, dy, hr, m, seco);  
        
    
    //printf("%lf minutes, %lf seconds\n", m, seco);
    //printf("\n");
    
    return -1;
}

struct load_avg pfs_load_avg(char *proc_dir)
{
   struct load_avg lavg = { 0 };
   char buf[128];
   int fd, read_sz;
   char loadAve[10000];
   char ladir[] = "/loadavg";
   
   strcpy(loadAve, proc_dir);
   strcat(loadAve, ladir);
   fd = open (loadAve, O_RDONLY);
   
   if (fd <= 2) {
       perror("error opening file");
   }
   
   read_sz = read(fd, buf, 128);
   int tokens = 0;
   char *next_tok = buf;
   char *curr_tok;

   while ((curr_tok = next_token(&next_tok, " ")) != NULL && (tokens != 3) ) {
       if (tokens == 0 ){
           lavg.one = atof(curr_tok);
       } else if (tokens == 1) {
           lavg.five = atof(curr_tok);
       } else if (tokens == 2) {
           lavg.fifteen = atof(curr_tok);
       }
       tokens++;
   }
   close(fd);

   return lavg;
}

double pfs_cpu_usage(struct cpu_stats *prev, struct cpu_stats *curr)
{
    double id1 =0;
    double id2 =0;
        double total1 = 0;
    double total2 = 0;
    int read_sz, fd;    
    double idlePer =0.0;
    char tewbuf[10000];

    char *next_tok;
    char punits[10000];
    char statBuf[1000];
    
    char pdir[] = "/stat";
    strcpy(punits, proc_dir);     
    strcat(punits, pdir);
    
    fd = open(punits, O_RDONLY);
    if (fd <= 2){
        perror("open");
        exit(0);
    }
    int ret = 0;
    int len =0;

    read_sz = read(fd, statBuf, 1000);
    next_tok = statBuf;
    int count = 0;
        char *fline = next_token(&next_tok, "\n");
    

    // double idiff = curr->idle - prev->idle;
    // double tdiff = curr->total - prev->total;
    // double cpuPer =0;
    
    // LOG("CPU usage check, idiff: %lf and tdiff: %lf\n",idiff, tdiff);
    // if (tdiff < 0 || idiff < 0){
    //         return 0.0;
    // }   

    // idlePer = idiff / tdiff;
    // cpuPer = 1 - idlePer;
    //  LOG("idel percent is %lf idlePer\n", idlePer);
    // return idlePer;
    // return idlePer*100;
    return 0.0;
}

struct mem_stats pfs_mem_usage(char *proc_dir )
{
    struct mem_stats mstats = { 0 };
    int fd;
    char buf[128];
    ssize_t read_sz;
    char memUsage[10000];
    char memdir[] = "/meminfo";
    strcpy(memUsage, proc_dir);
    strcat(memUsage, memdir);

    fd = open(memUsage, O_RDONLY);
    if (fd <= 2) {
        perror("open");
    }
    
    char tewbuf[10000];     
    read_sz = read(fd, tewbuf, 10000);
    char *next_tok = tewbuf;

    char *memRef = "MemTotal";
    char *avRef = "MemAvailable";
    bool totalNext = false;
    bool avNext = false;
    char av[64];
    char memT[64];
    double memTotf =0;
    double avail =0; 
    char *curr_tok; 

    /* Tokenize. Note that ' ,?!' will all be removed. */
    while ((curr_tok = next_token(&next_tok, "\t :\n")) != NULL) {
        if (totalNext == true) {
            strcpy(memT, curr_tok);
            LOG("\ntotla memory is %s\n", memT);
            totalNext = false;
        } else if(avNext == true) {
            strcpy(av, curr_tok);
            
            avNext = false; 
        }
         if (strcmp(curr_tok, avRef) == 0) {
            avNext = true;
         }
         if (strcmp(curr_tok, memRef) == 0) {        
                totalNext = true;
             }          

    }

    close(fd);
    
    //DO SOME CONTEXT CEHKEDING
    memTotf = atof(memT);
    avail = atof(av);
    double totconv = (memTotf * 1024) / 1073741824;
    double avconv =(memTotf*1024) - (avail * 1024) ;
    avconv = avconv / 1073741824;
    
    double memPer =0.0;
    LOG("memtotal is %lf and avaable is %lf", memTotf, avail);
    mstats.used =  avconv;// ((memTotf-avail)/ gig);
    mstats.total= totconv;
    return mstats;
}
struct task_stats *pfs_create_tstats()
{
    return NULL;
}

void pfs_destroy_tstats(struct task_stats *tstats)
{

}

int pfs_tasks(char *proc_dir, struct task_stats *tstats)
{
    return -1;
}

