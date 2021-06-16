Реализуйте функцию star, которая создаёт N+1 процессов (1 «центральный» и N «крайних») и посылает сообщение центральному процессу, который посылает сообщение всем остальным процессам и дожидается от них ответа, после чего это повторяется (всего M раз). После того, как все события получены, все процессы заканчивают работу.
Код(для VS19):
#include "mpi.h"
#include <iostream>
#include <stdio.h>
#define N 5
using namespace std;

int main(int argc, char* argv[])
{

	void star(int argc, char* argv[]);

	star(argc, argv);
	
	return 0;
}
void star(int argc, char* argv[])
{
	int ProcNum, ProcRank, RecvRank;
	int M = 1;
	int sbuf[N][N]=
	{
		{1,2,3,4,5},
	    {6,7,8,9,10},
	    {11,12,13,14,15},
	    {16,17,18,19,20},
	    {21,22,23,24,25}
	};
	int recvbuf[N];
	MPI_Status Status;
	MPI_Init(&argc, &argv);
	MPI_Comm_size(MPI_COMM_WORLD, &ProcNum);
	MPI_Comm_rank(MPI_COMM_WORLD, &ProcRank);
	for (int i = 0; i < M; i++)
	{
		if (ProcRank == 0)
		{ 
			cout << "SendMessage: " << endl;
			for (int j = 0; j < N; j++)
			{
				for (int t = 0; t < N; t++)
				{
					cout << sbuf[j][t] << "\t";
				}
				cout << endl;
			}
		}
		
		MPI_Scatter(sbuf, N, MPI_INT, recvbuf, N, MPI_INT, 0, MPI_COMM_WORLD);
		cout << "\nMessage from process 0 to " << ProcRank << ": ";
		for (int j = 0; j < N; j++)
		{
			cout <<"\t"<< recvbuf[j];
		}

	}

	MPI_Finalize();
}

  
  
 Результат:
  ![z_UYlUt6fUg](https://user-images.githubusercontent.com/61342960/122220524-11edc980-ceb9-11eb-85f2-e70954efbb79.jpg)
