#include <pthread.h>		
#include <time.h>			
#include <unistd.h>			
#include <semaphore.h>		
#include <stdlib.h>			
#include <stdio.h>			

pthread_t *Students;		
pthread_t TA;				

int cc = 0; //chair count
int ci = 0; // chair index


sem_t TAS; //TA_Sleep
sem_t ss; //student sem
sem_t cs[3]; //chairs sem
pthread_mutex_t ca; // chair access

void *TA_Activity();
void *Student_Activity(void *threadID);

int main()
{
	int nos;		// no of student
	int id;
	srand(time(NULL));

	sem_init(&TAS, 0, 0);
	sem_init(&ss, 0, 0);
	for(id = 0; id < 3; ++id)		
		sem_init(&cs[id], 0, 0);

	pthread_mutex_init(&ca, NULL);
	

		printf("Enter the number of student\n");
		scanf("%d",&nos);
		
		printf("Creating %d threads.\n", nos);
	
Students = (pthread_t*) malloc(sizeof(pthread_t)*nos);

	pthread_create(&TA, NULL, TA_Activity, NULL);	
	for(id = 0; id < nos; id++)
		pthread_create(&Students[id], NULL, Student_Activity,(void*) (long)id);

	pthread_join(TA, NULL);
	for(id = 0; id < nos; id++)
		pthread_join(Students[id], NULL);

	free(Students); 
	return 0;
}

void *TA_Activity()
{
	while(1)
	{
		sem_wait(&TAS);	
		printf("************TA has been awakened by a student*****************\n");

		while(1)
		{
			pthread_mutex_lock(&ca);
			if(cc == 0) 
			{
				pthread_mutex_unlock(&ca);
				break;
			}
			sem_post(&cs[ci]);
			cc--;
			printf("Student left chair. Remaining Chairs %d\n", 3 - cc);
			ci = (ci + 1) % 3;
			pthread_mutex_unlock(&ca);
		
			printf("\t TA is currently helping the student.\n");
			sleep(5);
			sem_post(&ss);
		}
	}
}

void *Student_Activity(void *threadID) 
{
	int ProgrammingTime;

	while(1)
	{
		printf("Student %ld is doing programming assignment.\n", (long)threadID);
		ProgrammingTime = rand() % 10 + 1;
		sleep(ProgrammingTime);

		printf("Student %ld needs help from the TA\n", (long)threadID);
		
		pthread_mutex_lock(&ca);
		int count = cc;
		pthread_mutex_unlock(&ca);

		if(count < 3)		
		{
			if(count == 0)	
				sem_post(&TAS);
			else
				printf("Student %ld sat on a chair waiting for the TA to finish. \n", (long)threadID);

			pthread_mutex_lock(&ca);
			int index = (ci + cc) % 3;
			cc++;
			printf("Student sat on chair.Chairs Remaining: %d\n", 3 - cc);
			pthread_mutex_unlock(&ca);

			sem_wait(&cs[index]);		
			printf("\t Student %ld is getting help from the TA. \n", (long)threadID);
			sem_wait(&ss);		
			printf("Student %ld left TA room.\n",(long)threadID);
		}
		else 
			printf("Student %ld will return at another time. \n", (long)threadID);
	}
}
