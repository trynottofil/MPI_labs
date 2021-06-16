# Лабораторная работа №5
Реализуйте на основе технологии MPI многопоточную программу с использованием групп процессов и коммуникаторов. Реализуйте тип "комплексная матрица". Напишите программу, которая осуществляет умножение А матриц размером N * N, методом Штрассена.

# Код(для VS19):
    #include "mpi.h"
    #include <iostream>
    #include <stdio.h>
    #include <locale.h>
    #include <time.h>
    #include <iomanip>
    #include <complex>
    #define N 4
    using namespace std;

    int** Create_Matrix(int rank)
    {
      int **mtrx = new int*[N];
      for (int i = 0; i < N; i++)
      {
        mtrx[i] = new int[N];
        for (int j = 0; j < N; j++)
        {
          mtrx[i][j] = rand() % 9 + rank;
          cout << mtrx[i][j];
        }
        cout << "\n";
      }
      return mtrx;
    }

    void Matrix_Multiply(complex <double> A[][N], complex <double> B[][N], complex <double> C[][N]) {  // A*B->C
      for (int i = 0; i < 2; i++) {
        for (int j = 0; j < 2; j++) {
          C[i][j] = 0;
          for (int t = 0; t < 2; t++) {
            C[i][j] = C[i][j] + A[i][t] * B[t][j];
          }
        }
      }
    }

    void Matrix_Add(int n, complex <double> X[][N], complex <double> Y[][N], complex <double> Z[][N]) { //A+B->Z
      for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
          Z[i][j] = X[i][j] + Y[i][j];
        }
      }
    }

    void Matrix_Sub(int n, complex <double> X[][N], complex <double> Y[][N], complex <double> Z[][N]) {//A-B->Z
      for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
          Z[i][j] = X[i][j] - Y[i][j];
        }
      }
    }

    void Print_Matrix(complex <double> m[N][N], int proc_rank)
    {
      cout << "Matrix (" << proc_rank << " process): " << endl;
      for (int i = 0; i < N; i++)
      {
        for (int j = 0; j < N; j++)
        {
          cout << setw(12) << m[i][j];
        }
        cout << endl;
      }
    }

    void Strassen(int n, complex <double> A[][N], complex <double> B[][N], complex <double> C[][N]) {
      complex <double> A11[N][N], A12[N][N], A21[N][N], A22[N][N];
      complex <double> B11[N][N], B12[N][N], B21[N][N], B22[N][N];
      complex <double> C11[N][N], C12[N][N], C21[N][N], C22[N][N];
      complex <double> M1[N][N], M2[N][N], M3[N][N], M4[N][N], M5[N][N], M6[N][N], M7[N][N];
      complex <double> AA[N][N], BB[N][N];

      if (n == 2) {
        Matrix_Multiply(A, B, C);
      }
      else {
        for (int i = 0; i < n / 2; i++) {
          for (int j = 0; j < n / 2; j++) {
            A11[i][j] = A[i][j];
            A12[i][j] = A[i][j + n / 2];
            A21[i][j] = A[i + n / 2][j];
            A22[i][j] = A[i + n / 2][j + n / 2];

            B11[i][j] = B[i][j];
            B12[i][j] = B[i][j + n / 2];
            B21[i][j] = B[i + n / 2][j];
            B22[i][j] = B[i + n / 2][j + n / 2];
          }
        }

        //M1 = (A0 + A3) × (B0 + B3)
        Matrix_Add(n / 2, A11, A22, AA);
        Matrix_Add(n / 2, B11, B22, BB);
        Strassen(n / 2, AA, BB, M1);

        //M2 = (A2 + A3) × B0
        Matrix_Add(n / 2, A21, A22, AA);
        Strassen(n / 2, AA, B11, M2);

        //M3 = A0 × (B1 - B3)
        Matrix_Sub(n / 2, B12, B22, BB);
        Strassen(n / 2, A11, BB, M3);

        //M4 = A3 × (B2 - B0)
        Matrix_Sub(n / 2, B21, B11, BB);
        Strassen(n / 2, A22, BB, M4);

        //M5 = (A0 + A1) × B3
        Matrix_Add(n / 2, A11, A12, AA);
        Strassen(n / 2, AA, B22, M5);

        //M6 = (A2 - A0) × (B0 + B1)
        Matrix_Sub(n / 2, A21, A11, AA);
        Matrix_Add(n / 2, B11, B12, BB);
        Strassen(n / 2, AA, BB, M6);

        //M7 = (A1 - A3) × (B2 + B3)
        Matrix_Sub(n / 2, A12, A22, AA);
        Matrix_Add(n / 2, B21, B22, BB);
        Strassen(n / 2, AA, BB, M7);

        //C0 = M1 + M4 - M5 + M7
        Matrix_Add(n / 2, M1, M4, AA);
        Matrix_Sub(n / 2, M7, M5, BB);
        Matrix_Add(n / 2, AA, BB, C11);

        //C1 = M3 + M5
        Matrix_Add(n / 2, M3, M5, C12);

        //C2 = M2 + M4
        Matrix_Add(n / 2, M2, M4, C21);

        //C3 = M1 - M2 + M3 + M6
        Matrix_Sub(n / 2, M1, M2, AA);
        Matrix_Add(n / 2, M3, M6, BB);
        Matrix_Add(n / 2, AA, BB, C22);

        //Set the result to C[][N]
        for (int i = 0; i < n / 2; i++) {
          for (int j = 0; j < n / 2; j++) {
            C[i][j] = C11[i][j];
            C[i][j + n / 2] = C12[i][j];
            C[i + n / 2][j] = C21[i][j];
            C[i + n / 2][j + n / 2] = C22[i][j];
          }
        }
        //Print_Matrix(C);
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
      MPI_Status Status;
      MPI_Init(&argc, &argv);
      MPI_Comm_size(MPI_COMM_WORLD, &ProcNum);
      MPI_Comm_rank(MPI_COMM_WORLD, &ProcRank);
      MPI_Group new_group;
      MPI_Group original_group;
      int ranks_for_new_group[10] = { 0,1,2,3,4,5,6,7,8,9 };
      MPI_Comm_group(MPI_COMM_WORLD, &original_group);
      MPI_Group_incl(original_group, 10, ranks_for_new_group, &new_group);
      MPI_Comm new_comm;
      MPI_Comm_create(MPI_COMM_WORLD, new_group, &new_comm);

      complex <double> m1[N][N];
      complex <double> m2[N][N];
      complex <double> c[N][N];
      complex <double> res[N][N];
      complex <double> buff[N][N];
      for (int i = 0; i < N; i++)
      {
        for (int j = 0; j < N; j++)
        {
          complex <double> x(i + ProcRank, j + ProcRank);
          complex <double> y(i + ProcRank - 1, j + ProcRank - 1);
          m1[i][j] = x;
          m2[i][j] = y;
          c[i][j] = 0;
          buff[i][j] = 0;
          res[i][j] = 0;
        }
      }
      if (ProcRank == 0)
      {
        Print_Matrix(m1, ProcRank);
        Print_Matrix(m2, ProcRank);
      }

      MPI_Barrier(MPI_COMM_WORLD);

      if (ProcRank == 1)
      {
        Print_Matrix(m1, ProcRank);
        Print_Matrix(m2, ProcRank);
      }

      MPI_Datatype type_1;
      MPI_Type_contiguous(N*N, MPI_DOUBLE_COMPLEX, &type_1);
      MPI_Type_commit(&type_1);

      Strassen(N, m1, m2, c);

      for (int i = 0; i < ProcNum; i++)
      {
        if ((ProcRank == i) && (i % 2 != 0) && (ProcRank != 0))
        {
          MPI_Send(c, 1, type_1, ProcRank - 1, 1, new_comm);
        }
        else if ((ProcRank == i) && (i % 2 == 0))
        {
          MPI_Recv(buff, 1, type_1, ProcRank + 1, MPI_ANY_TAG, new_comm, &Status);
        }
      }

      MPI_Barrier(MPI_COMM_WORLD);

      if (ProcRank % 2 == 0)
      {
        Strassen(N, c, buff, res); // 0  2  4  6  8 
        Print_Matrix(res, ProcRank);
      }

      MPI_Barrier(MPI_COMM_WORLD);


      for (int i = 0; i < ProcNum; i++)
      {
        if (((ProcRank == i) && (i == 2)) || ((ProcRank == i)&&(ProcRank == 6)))
        {
          MPI_Send(res, 1, type_1, ProcRank - 2, 1, new_comm);
        }
        else if (((ProcRank == i) && (i == 0)) || ((ProcRank == i)&&(i == 4)))
        {
          MPI_Recv(buff, 1, type_1, ProcRank + 2, MPI_ANY_TAG, new_comm, &Status);
          Strassen(N, res, buff, c);// 0 4 8
          Print_Matrix(c, ProcRank);
        }
      }

      MPI_Barrier(MPI_COMM_WORLD);

      if (ProcRank == 4)
      {
        MPI_Send(c, 1, type_1, 0, 1, new_comm);
      }
      if (ProcRank == 0)
      {
        MPI_Recv(buff, 1, type_1, 4, MPI_ANY_TAG, new_comm, &Status);
        Strassen(N, c, buff, res);
        Print_Matrix(res, ProcRank);
      }

      MPI_Barrier(MPI_COMM_WORLD);

      if (ProcRank == 8)
      {
        MPI_Send(res, 1, type_1, 0, 1, new_comm);
      }

      if (ProcRank == 0)
      {
        MPI_Recv(c, 1, type_1, 8, MPI_ANY_TAG, new_comm, &Status);
        Strassen(N, res, c, buff);
        cout << "Result ";
        Print_Matrix(buff, ProcRank);
      }

      MPI_Type_free(&type_1);
      MPI_Group_free(&new_group);
      MPI_Comm_free(&new_comm);

      MPI_Finalize();
    }

# Результат:
![lab5](https://user-images.githubusercontent.com/61342960/122224525-dd7c0c80-cebc-11eb-942a-029e49c4af74.png)
