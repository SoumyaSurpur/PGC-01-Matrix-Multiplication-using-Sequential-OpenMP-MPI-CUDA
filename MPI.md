# PART C - MPI Distributed Cluster & Matrix Multiplication

## 1. Cluster Topology & Network Architecture

The distributed computing experiment was deployed across a 4-node Ubuntu virtual machine cluster interconnected via a private host-only virtual network on subnet `192.168.190.0/24`:

| Node Name | Hostname | IP Address | Cluster Role | Task Partitioning |
| :--- | :--- | :--- | :--- | :--- |
| **Master** | `master` | `192.168.190.128` | Rank 0 (Coordinator) | Manages cluster; computes rows 0 – 999 |
| **Worker 1** | `worker1` | `192.168.190.129` | Rank 1 (Worker) | Computes rows 1000 – 1999 |
| **Worker 2** | `worker2` | `192.168.190.130` | Rank 2 (Worker) | Computes rows 2000 – 2999 |
| **Worker 3** | `worker3` | `192.168.190.131` | Rank 3 (Worker) | Computes rows 3000 – 3999 |

---

## 2. Network Connectivity

The Master VM was used to verify communication with all Worker VMs using `ping`.

The connectivity test was successful with **0% packet loss** for all three workers.

<img width="616" height="543" alt="mpi_ping" src="https://github.com/user-attachments/assets/fbd17b6d-d17d-46fe-b6df-42bf823289b1" />

---

## 3. SSH Configuration

OpenSSH was configured on the Worker VMs to allow remote access from the Master.

The SSH service was verified to be active and running on Worker1, Worker2, and Worker3.

<img width="701" height="528" alt="sshworker1_mpi" src="https://github.com/user-attachments/assets/dd4bc6c0-cbac-4560-a347-a4ce607e8107" />

<img width="693" height="534" alt="sshworker2_mpi" src="https://github.com/user-attachments/assets/186d252c-896a-4644-8514-c96fa2672829" />

<img width="735" height="513" alt="sshworker3_mpi" src="https://github.com/user-attachments/assets/8674cd35-41a1-411a-a521-f9fce359064b" />

---

## 4. SSH Communication Test

The Master successfully connected to Worker1, Worker2 and Worker3 using SSH.

The `hostname` command confirmed that the connection was established with the correct worker node.

<img width="851" height="404" alt="worker1_mpi" src="https://github.com/user-attachments/assets/136b8087-8bfb-46ec-8eb9-eabf7778544e" />
<img width="736" height="401" alt="worker2_mpi" src="https://github.com/user-attachments/assets/8cf25177-f7b4-44c8-a82c-b324709ab4df" />
<img width="694" height="668" alt="WhatsApp Image 2026-09-25 at 9 20 28 AM" src="https://github.com/user-attachments/assets/23646b97-841d-41d9-9db1-493a6bfa8767" />

---

## 5. Passwordless SSH Key Exchange

To allow `mpirun` to launch processes on remote nodes without human password interaction:

1. **Key Generation on Master**: Generated 3072-bit RSA keypair without passphrase:
   ```bash
   ssh-keygen -t rsa
   ```
   <img width="654" height="406" alt="WhatsApp Image 2026-09-25 at 9 20 28 AM (3)" src="https://github.com/user-attachments/assets/4e6d67e4-bd31-49d2-8667-575e39b56adc" />


2. **Public Key Distribution**: Installed Master public key into all worker `authorized_keys`:
   ```bash
   ssh-copy-id worker1@worker1
   ssh-copy-id worker2@worker2
   ssh-copy-id worker3@worker3
   ```
  <img width="885" height="744" alt="WhatsApp Image 2026-09-25 at 9 28 15 AM (1)" src="https://github.com/user-attachments/assets/5983be58-254f-4922-a9f8-8b5f07a7fa1f" />


3. **Passwordless Verification**: Verified remote command execution without password prompts:
   ```bash
   ssh worker1 hostname
   ssh worker2 hostname
   ssh worker3 hostname
   ```
  <img width="463" height="166" alt="WhatsApp Image 2026-09-25 at 9 35 17 AM (1)" src="https://github.com/user-attachments/assets/a5eb4edb-3c03-44da-ad19-9aba1d7e8ab4" />

---

## 6. MPI Installation & Cluster Toolchain Verification

 MPI was installed across all cluster nodes using the Ubuntu package manager:

```bash
sudo apt update
sudo apt install openmpi-bin libopenmpi-dev -y
```

### Installation & Verification on Master Node
Verified `mpicc` compiler wrapper, MPI 5.0.10 info, and local process launcher:
<img width="807" height="347" alt="WhatsApp Image 2026-09-25 at 9 51 29 AM (3)" src="https://github.com/user-attachments/assets/2c1e64a2-0138-49d2-9cf2-82cb9b04e137" />
<img width="790" height="542" alt="WhatsApp Image 2026-09-25 at 9 51 29 AM (5)" src="https://github.com/user-attachments/assets/b858fc21-1e0f-4fea-847e-c03e540d2ea2" />


### Installation & Verification on Worker Nodes
Verified MPI installation on Worker 1, Worker 2, and Worker 3:
<img width="849" height="832" alt="WhatsApp Image 2026-09-25 at 9 51 29 AM (4)" src="https://github.com/user-attachments/assets/a31d83b4-27f1-4d2f-adcc-96cc4c43f2ad" />

#### Worker 1 Verify
<img width="854" height="880" alt="WhatsApp Image 2026-09-25 at 9 51 30 AM (3)" src="https://github.com/user-attachments/assets/89f24a94-bff9-46d2-8e9b-68b570484971" />

#### Worker 2 Verify
<img width="796" height="832" alt="WhatsApp Image 2026-09-25 at 9 51 30 AM (4)" src="https://github.com/user-attachments/assets/0346094b-f1b3-409f-8cd3-8698c64b5d28" />

#### Worker 3 Verify
<img width="839" height="846" alt="WhatsApp Image 2026-09-25 at 9 51 30 AM (5)" src="https://github.com/user-attachments/assets/03807a69-b09f-46f3-be9a-63004dd94ca1" />

---

## 7. Hostfile Configuration & Distributed Message Passing Validation

### 7.1 Hostfile Configuration
Created `hosts` on Master defining the cluster slots:
```text
master slots=1
worker1 slots=1
worker2 slots=1
worker3 slots=1
```

Tested remote process launch across all four nodes:
```bash
mpirun -np 4 --hostfile hosts hostname
```

### 7.2 Point-to-Point Message Passing Verification (`mpi_send_recv.c`)
Before running dense matrix multiplication, a point-to-point program was compiled and distributed across nodes using `scp`:
<img width="848" height="404" alt="WhatsApp Image 2026-09-25 at 10 07 01 AM (5)" src="https://github.com/user-attachments/assets/c14b97a3-c40e-4259-a4c5-1e0ece00136d" />


Verified binary availability across all three worker home directories:
| Worker 1 Executable | Worker 2 Executable | Worker 3 Executable |
| :---: | :---: | :---: |
| <img width="300" height="132" alt="WhatsApp Image 2026-09-25 at 10 07 01 AM (6)" src="https://github.com/user-attachments/assets/6c63bb76-ce9f-4527-a5bd-74a1241ced81" /> | <img width="300" height="132" alt="WhatsApp Image 2026-09-25 at 10 07 01 AM (6)" src="https://github.com/user-attachments/assets/83bd2fd7-324f-4e8a-884c-cc51c089ce0f" /> | <img width="300" height="132" alt="WhatsApp Image 2026-09-25 at 10 07 01 AM (7)" src="https://github.com/user-attachments/assets/7714ef27-970c-475e-a727-cf3a620a152d" /> |

Executed the distributed message-passing program across 4 nodes:
```bash
env -u DISPLAY mpirun -np 4 --hostfile hosts sh -c '$HOME/mpi_send_recv'
```
<img width="807" height="149" alt="WhatsApp Image 2026-09-25 at 10 14 19 AM (1)" src="https://github.com/user-attachments/assets/bedbea94-ebfe-4451-b951-a00396189938" />


```text
Rank 0 is running on master
Rank 0 on master: Sending A = 10 to Rank 1
Rank 2 is running on worker2
Rank 1 is running on worker1
Rank 3 is running on worker3
Rank 1 on worker1: Received A = 10 from Rank 0
```

---

## 8. Distributed Matrix Multiplication Program

The dense $4000 \times 4000$ matrix multiplication distributes computation across the 4 nodes:
* `MPI_Scatter`: Partitions Matrix $A$ into four 1000-row slices ($32\text{ MB}$ each) from Rank 0 to all workers.
* `MPI_Bcast`: Broadcasts the entire Matrix $B$ ($128\text{ MB}$) to every rank.
* `MPI_Gather`: Assembles computed row slices of Matrix $C$ back onto Rank 0.

```c
#include <stdio.h>
#include <stdlib.h>
#include <mpi.h>
#include <unistd.h>

#define N 4000

int main(int argc, char *argv[])
{
    int rank, size;
    int i, j, k;
    int rows_per_process;
    char hostname[256];

    double *A = NULL;
    double *B = NULL;
    double *C = NULL;
    double *local_A;
    double *local_C;

    double start, end;

    MPI_Init(&argc, &argv);

    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Comm_size(MPI_COMM_WORLD, &size);

    gethostname(hostname, sizeof(hostname));

    if (N % size != 0)
    {
        if (rank == 0)
            printf("Matrix size must be divisible by number of processes.\n");

        MPI_Finalize();
        return 0;
    }

    rows_per_process = N / size;

    local_A = (double *)malloc(rows_per_process * N * sizeof(double));
    local_C = (double *)malloc(rows_per_process * N * sizeof(double));
    B = (double *)malloc(N * N * sizeof(double));

    if (rank == 0)
    {
        A = (double *)malloc(N * N * sizeof(double));
        C = (double *)malloc(N * N * sizeof(double));

        printf("Initializing %d x %d matrices...\n", N, N);

        for (i = 0; i < N; i++)
        {
            for (j = 0; j < N; j++)
            {
                A[i * N + j] = 1.0;
                B[i * N + j] = 1.0;
                C[i * N + j] = 0.0;
            }
        }
    }

    MPI_Barrier(MPI_COMM_WORLD);
    start = MPI_Wtime();

    MPI_Scatter(A, rows_per_process * N, MPI_DOUBLE,
                local_A, rows_per_process * N, MPI_DOUBLE,
                0, MPI_COMM_WORLD);

    MPI_Bcast(B, N * N, MPI_DOUBLE, 0, MPI_COMM_WORLD);

    printf("Rank %d on %s computing %d rows\n", rank, hostname, rows_per_process);

    for (i = 0; i < rows_per_process; i++)
    {
        for (j = 0; j < N; j++)
        {
            local_C[i * N + j] = 0.0;

            for (k = 0; k < N; k++)
            {
                local_C[i * N + j] += local_A[i * N + k] * B[k * N + j];
            }
        }
    }

    MPI_Gather(local_C, rows_per_process * N, MPI_DOUBLE,
               C, rows_per_process * N, MPI_DOUBLE,
               0, MPI_COMM_WORLD);

    MPI_Barrier(MPI_COMM_WORLD);
    end = MPI_Wtime();

    if (rank == 0)
    {
        printf("\nMPI Matrix Multiplication Completed\n");
        printf("Matrix Size = %d x %d\n", N, N);
        printf("Number of MPI Processes = %d\n", size);
        printf("Execution Time = %f seconds\n", end - start);
        printf("Verification C[0][0] = %.2f\n", C[0]);

        free(A);
        free(C);
    }

    free(B);
    free(local_A);
    free(local_C);

    MPI_Finalize();
    return 0;
}
```

---

## 9. Compilation, Staging & Cluster Execution

```bash
# 1. Compile on Master
mpicc -O2 matrix_mpi.c -o matrix_mpi

# 2. Stage executable to all Worker nodes
scp matrix_mpi worker1:~/matrix_mpi
scp matrix_mpi worker2:~/matrix_mpi
scp matrix_mpi worker3:~/matrix_mpi

# 3. Launch distributed job across cluster
mpirun -np 4 --hostfile hosts sh -c '$HOME/matrix_mpi'
```

---

## 10. Final Benchmark Result & Verification

```text
Initializing 4000 x 4000 matrices...
Rank 0 on master computing 1000 rows
Rank 1 on worker1 computing 1000 rows
Rank 2 on worker2 computing 1000 rows
Rank 3 on worker3 computing 1000 rows

MPI Matrix Multiplication Completed
Matrix Size = 4000 x 4000
Number of MPI Processes = 4
Execution Time = 92.979510 seconds
Verification C[0][0] = 4000.00
```

### Metrics Summary:
* **Execution Time ($T_{mpi}$)**: **`92.979510 seconds`**
* **Sequential Baseline ($T_{seq}$)**: `244.120000 seconds`
* **Speedup over Sequential**: $\frac{244.120000}{92.979510} \approx \mathbf{2.63\times}$
* **Parallel Efficiency**: $\frac{2.63}{4} \approx \mathbf{65.8\%}$
* **Verification Status**: $C[0][0] = 4000.00$ (**PASSED**)
