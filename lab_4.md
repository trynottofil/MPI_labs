# Лабораторная работа №4
Реализуйте тип "длинное целое". Напишите программу, которая осуществляет умножение А целых чисел заданной длины.

# Код(для VS19):
    #include "mpi.h"
    #include <iostream>
    #include <stdio.h>
    #include <locale.h>
    #include <time.h>

    #define N 5  

    using namespace std;

    int *create_buff(int length)
    {
      int *buff = new int[length];
      for (int i = length - 1; i >= 0; i--)
      {
        buff[i] = 0;
      }
      return buff;
    }
    int *create_num(int ProcRank)
    {
      int *num = new int[N];
      for (int i = N - 1; i >= 0; i--)
      {
        num[i] = ProcRank + (rand() % 4 + 1);
      }
      return num;
    }
    void print_num(int *num, int length)
    {
      for (int i = length - 1; i >= 0; i--)
      {
        cout << num[i];
      }
    }
    void mult(int* num_1, int* num_2, int* res, int size_num_1, int size_num_2)
    {
      for (int ix = 0; ix < size_num_1; ix++)
      {
        for (int jx = 0; jx < size_num_2; jx++)
        {
          res[ix + jx] += num_1[ix] * num_2[jx];
        }
      }

      for (int ix = 0; ix < size_num_1 + size_num_2; ix++)
      {
        res[ix + 1] += res[ix] / 10;
        res[ix] %= 10;
      }
    }

    int main(int argc, char* argv[])
    {
      void star(int argc, char* argv[]);

      star(argc, argv);

      return 0;
    }
    void star(int argc, char* argv[])
    {
      int ProcNum, ProcRank, RecvRank;
      int arr[5] = {0, 1, 2, 3, 4};
      MPI_Status Status;
      MPI_Init(&argc, &argv);
      MPI_Comm_size(MPI_COMM_WORLD, &ProcNum);
      MPI_Comm_rank(MPI_COMM_WORLD, &ProcRank);
      srand(time(NULL));
      cout << "\nNum_1 from process " << ProcRank << " = "; 
      int *num_1 = create_num(ProcRank);
      int *num_2 = create_num(ProcRank);
      print_num(num_1, N);

      MPI_Barrier(MPI_COMM_WORLD);

      cout << "\nNum_2 from process " << ProcRank << " = ";

      print_num(num_2, N);

      MPI_Barrier(MPI_COMM_WORLD);

      int *buff_1 = create_buff(N * 2);

      int *buff_2 = create_buff(N * 2 * 2);

      int *buff_3 = create_buff(N * 2 * 2 * 2);

      int *res_1 = create_buff(N * 2);

      int *res_2 = create_buff(N * 2 * 2);

      int *res_3 = create_buff(N * 2 * 2 * 2 + N * 2);

      MPI_Datatype type_1; 
      MPI_Type_contiguous(N*2, MPI_INT, &type_1);
      MPI_Type_commit(&type_1);

      MPI_Datatype type_2; 
      MPI_Type_contiguous(N * 2 * 2, MPI_INT, &type_2);
      MPI_Type_commit(&type_2);

      mult(num_1, num_2, res_1, N, N);
      MPI_Barrier(MPI_COMM_WORLD);
      cout <<"\nProcces "<<ProcRank << ": num_1 * num_2 = ";

      print_num(res_1, N * 2);

      MPI_Barrier(MPI_COMM_WORLD);

      for (int i = 0; i < 4; i++) 
      {
        if ((ProcRank == i) && (i % 2 != 0) && (ProcRank != 0))
        {
          MPI_Send(res_1, 1, type_1, ProcRank - 1, 1, MPI_COMM_WORLD);
        }
        else if ((ProcRank == i) && (i % 2 == 0))
        {
          MPI_Recv(buff_1, 1, type_1, ProcRank + 1, MPI_ANY_TAG, MPI_COMM_WORLD, &Status);
        }
      }


      MPI_Barrier(MPI_COMM_WORLD);

      if (ProcRank == 0 || ProcRank == 2)
      {
        mult(res_1, buff_1, res_2, N * 2, N * 2);
        if (ProcRank == 0)
        {
          cout << endl;
          print_num(res_1, N * 2);
          cout << " * ";
          print_num(buff_1, N * 2);
          cout << " = ";
          print_num(res_2, N * 2 * 2);
        }
        if (ProcRank == 2)
        {
          cout << endl;
          print_num(res_1, N * 2);
          cout << " * ";
          print_num(buff_1, N * 2);
          cout << " = ";
          print_num(res_2, N * 2 * 2);
        }

      }
      MPI_Barrier(MPI_COMM_WORLD);

      if (ProcRank == 2)
      {
        MPI_Send(res_2, 1, type_2, 0, 0 , MPI_COMM_WORLD);
      }
      if (ProcRank == 0)
      {
        MPI_Recv(buff_2, 1, type_2, 2, MPI_ANY_TAG, MPI_COMM_WORLD, &Status);
      }

      MPI_Barrier(MPI_COMM_WORLD);

      if (ProcRank == 0)
      {
        mult(res_2, buff_2, buff_3, N * 2 * 2, N * 2 * 2);
        cout << endl;
        print_num(res_2, N * 2 * 2);
        cout << " * ";
        print_num(buff_2, N * 2 * 2);
        cout << " = ";
        print_num(buff_3, N * 2 * 2 * 2);
      }

      MPI_Barrier(MPI_COMM_WORLD);

      if (ProcRank == 4)
      {
        MPI_Send(res_1, 1, type_1, 0, 0, MPI_COMM_WORLD);
      }
      else if (ProcRank == 0)
      {
        MPI_Recv(res_1, 1, type_1, 4, MPI_ANY_TAG, MPI_COMM_WORLD, &Status);

        mult(buff_3, res_1, res_3, N * 2 * 2 * 2, N * 2);

        cout << "\nResult:\n";
        print_num(buff_3, N * 2 * 2 *2);
        cout << " * ";
        print_num(res_1, N * 2);
        cout << " = ";
        print_num(res_3, N * 2 * 2 * 2 + N * 2);
      }
      MPI_Barrier(MPI_COMM_WORLD);

      delete[] num_1;
      delete[] buff_1;
      delete[] res_1;
      delete[] num_2;
      delete[] buff_2;
      delete[] res_2;
      delete[] buff_3;
      delete[] res_3;
      MPI_Type_free(&type_1);
      MPI_Type_free(&type_2);

      MPI_Finalize();
    }
# Результат:
  ![z_UYlUt6fUg](https://user-images.githubusercontent.com/61342960/122222864-567a6480-cebb-11eb-84f4-556e08a26839.jpg)
