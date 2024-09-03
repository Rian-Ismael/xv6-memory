## xv6: Gerenciamento de Memória e Page Tables

## Capítulo 3: Page Tables e Gerenciamento de Memória

### 3.1 Hardware de Paginação

#### Estrutura das Page Tables

No xv6, as page tables são usadas para mapear endereços virtuais em endereços físicos. A arquitetura RISC-V, usada pelo xv6, implementa uma paginação em três níveis chamada Sv39. Isso significa que o processo de tradução de endereços envolve três níveis de page tables.

Page Table Entries (PTEs): Cada nível da page table contém entradas de page table (PTEs), que apontam para o próximo nível da page table ou para a página física final. Cada PTE no xv6 é um número de 64 bits, dos quais apenas alguns bits são usados para armazenar o endereço físico, enquanto os demais bits são usados para permissões e flags de controle.
Tamanho e Estrutura dos Endereços
Endereço Virtual: 64 bits, mas no modelo Sv39, apenas os 39 bits inferiores são usados para endereçamento, com os 25 bits superiores sendo extensões de sinal (sign extension).
Estrutura do Endereço Virtual: No Sv39, um endereço virtual é dividido em partes para acessar as page tables:
Nível 1 (VPN[2]): Bits 30-38
Nível 2 (VPN[1]): Bits 21-29
Nível 3 (VPN[0]): Bits 12-20
Offset: Bits 0-11
Essas partes são usadas para indexar as page tables em cada nível.
