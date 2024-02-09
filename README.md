/**
 * Grupo:
 * Mário Narcizo dos Anjos Júnior 
 * Luan Prata Mendonça 
 * Victória Moura Santos 
 */
 
// 
#include <stdio.h>
#include <string.h>  
#include <mpi.h>     


typedef struct Relogio { 
   
   int p[3];
} Relogio;

void print_relogio(int pid, Relogio * clock) {
   
   printf("Process: %d, Relogio: (%d, %d, %d)\n", pid, clock->p[0], clock->p[1], clock->p[2]);
}

void Evento(int pid, Relogio *clock){
   
   clock->p[pid]++;   
   
   print_relogio(pid, clock);
}

void Envio(int pid, int dest, Relogio *clock) {
   
   clock->p[pid]++;
   
   MPI_Send(clock, sizeof(Relogio), MPI_BYTE, dest, dest, MPI_COMM_WORLD);
   print_relogio(pid, clock);
}

void Recebe(int pid, int source, Relogio*clock) {
   
   int size, i;
   
   Relogio clock_received;
   
   MPI_Recv(&clock_recebe, sizeof(Relogio), MPI_BYTE, source, MPI_ANY_TAG, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
   
   // printf("Clock recebido: (%d, %d, %d)\n", clock_recebe.p[0], clock_recebe.p[1], clock_recebe.p[2]);
   
   MPI_Comm_size(MPI_COMM_WORLD, &size);
   
   clock->p[pid]++;
   
   for (i = 0; i < size; i++) {
      if (i != pid && clock->p[i] < clock_recebe.p[i]) {
         clock->p[i] = clock_recebe.p[i];
      }
   }
   
   print_relogio(pid, clock);
}

// Representa o processo de rank 0
void process0(){
   
   int pid;               
   
   MPI_Comm_rank(MPI_COMM_WORLD, &pid);
   
   Clock clock = {{0,0,0}};
   
   Evento(pid, &clock);           // a
   Envio(pid, 1, &clock);         // b
   Recebe(pid, 1, &clock);      // c
   Envio(pid, 2, &clock);         // d
   Recebe(pid, 2, &clock);      // e
   Envio(pid, 1, &clock);         // f
   Evento(pid, &clock);           // g
}

// Representa o processo de rank 1
void process1(){
   
   int pid;               
   
   MPI_Comm_rank(MPI_COMM_WORLD, &pid);
   
   Relogio clock = {{0,0,0}};
   
   Envio(pid, 0, &clock);         // h
   Recebe(pid, 0, &clock);      // i
   Recebe(pid, 0, &clock);      // j
}

// Representa o processo de rank 2
void process2(){
   
   int pid;               
   
   MPI_Comm_rank(MPI_COMM_WORLD, &pid);
   
   Relogio clock = {{0,0,0}};
   
   Evento(pid, &clock);           // k
   Envio(pid, 0, &clock);         // l
   Recebe(pid, 0, &clock);      // m
}

int main(void) {
   
   int my_rank;               

   MPI_Init(NULL, NULL); 
   MPI_Comm_rank(MPI_COMM_WORLD, &my_rank); 

   if (my_rank == 0) { 
      process0();
   } else if (my_rank == 1) {  
      process1();
   } else if (my_rank == 2) {  
      process2();
   }

   /* Finaliza MPI */
   MPI_Finalize(); 

   return 0;
}  /* main */