## xv6: Gerenciamento de Memória e Page Tables

## Capítulo 3: Page Tables e Gerenciamento de Memória

### 3.1 Hardware de Paginação

#### Estrutura das Page Tables

No xv6, as page tables são usadas para mapear endereços virtuais em endereços físicos. A arquitetura RISC-V, usada pelo xv6, implementa uma paginação em três níveis chamada Sv39. Isso significa que o processo de tradução de endereços envolve três níveis de page tables.

Page Table Entries (PTEs): Cada nível da page table contém entradas de page table (PTEs), que apontam para o próximo nível da page table ou para a página física final. Cada PTE no xv6 é um número de 64 bits, dos quais apenas alguns bits são usados para armazenar o endereço físico, enquanto os demais bits são usados para permissões e flags de controle.

#### Tamanho e Estrutura dos Endereços

Endereço Virtual: 64 bits, mas no modelo Sv39, apenas os 39 bits inferiores são usados para endereçamento, com os 25 bits superiores sendo extensões de sinal (sign extension).
Estrutura do Endereço Virtual: No Sv39, um endereço virtual é dividido em partes para acessar as page tables:
- Nível 1 (VPN[2]): Bits 30-38 *[Esta é a parte mais significativa do Número de Página Virtual. Ela é usada para indexar a primeira camada da tabela de páginas (também chamada de L1). Os bits 30-38 do endereço virtual são usados para acessar a entrada correspondente na tabela de páginas de nível 1.]*
- Nível 2 (VPN[1]): Bits 21-29 *[Esta é a parte intermediária do Número de Página Virtual. Ela é usada para indexar a segunda camada da tabela de páginas (L2). Os bits 21-29 do endereço virtual são extraídos e utilizados para acessar a entrada correspondente na tabela de páginas de nível 2.]*
- Nível 3 (VPN[0]): Bits 12-20 *[Esta é a parte menos significativa do Número de Página Virtual. Ela é usada para indexar a terceira e última camada da tabela de páginas (L3). Os bits 12-20 do endereço virtual são utilizados para acessar a entrada final que aponta para o endereço físico correspondente ou para outra tabela de página, dependendo da estrutura hierárquica.]*

- Offset: Bits 0-11

Essas partes são usadas para indexar as page tables em cada nível.

#### Cálculos da Tradução de Endereços

Índice do Nível 1: Para obter o índice do primeiro nível da page table, extrai-se o campo VPN[2] (bits 30-38) do endereço virtual. Esse índice é usado para acessar a PTE correspondente na page table do primeiro nível.

*Cálculo*:
Pegue o endereço virtual.
Desloque 30 bits à direita.
Use os 9 bits resultantes para indexar a PTE na page table de nível 1.
Índice do Nível 2: Similarmente, o índice para o segundo nível é obtido extraindo VPN[1] (bits 21-29) do endereço virtual.

*Cálculo*:
Pegue o endereço virtual.
Desloque 21 bits à direita.
Use os 9 bits resultantes para indexar a PTE na page table de nível 2.
Índice do Nível 3: Finalmente, o índice para o terceiro nível é obtido extraindo VPN[0] (bits 12-20) do endereço virtual.

*Cálculo*:
Pegue o endereço virtual.
Desloque 12 bits à direita.
Use os 9 bits resultantes para indexar a PTE na page table de nível 3.
Offset: Os 12 bits mais baixos do endereço virtual (bits 0-11) são usados como um offset dentro da página física final para acessar o dado específico.

*Cálculo*:
O offset é simplesmente a parte inferior do endereço virtual e é adicionado ao endereço base da página física.
Exemplo de Cálculo Completo

Esses valores de índice são usados para navegar nas page tables e encontrar o endereço físico correspondente.

### 3.2 Espaço de Endereço do Kernel

#### Mapas de Memória do Kernel

O kernel do xv6 mantém um mapeamento direto entre endereços virtuais e físicos para simplificar o acesso à memória física. Isso significa que uma porção dos endereços virtuais no espaço do kernel mapeia diretamente para os mesmos endereços físicos.

Exemplo: Um endereço virtual 0x80000000 pode mapear diretamente para o endereço físico 0x00000000.
Esse mapeamento direto é crucial para operações de baixo nível, como a manipulação de dispositivos de hardware.

#### Guard Pages e Proteção de Memória
Para proteger contra estouros de pilha e outros erros de memória, o xv6 usa guard pages. Essas páginas são intencionalmente deixadas não mapeadas, ou seja, qualquer tentativa de acesso a elas gera uma falha de página.

Exemplo: Se a pilha do kernel crescer demais e tentar acessar uma guard page, o sistema detecta isso e pode encerrar a operação ou realizar alguma forma de tratamento de erro.

### 3.3 Código: Criando um Espaço de Endereço
#### Função walk e Navegação nas Page Tables
A função walk é utilizada para percorrer a hierarquia de page tables e encontrar a entrada correspondente a um determinado endereço virtual.

#### Uso de walk:
Navega pelos níveis 1, 2 e 3 da page table.
Retorna a entrada da page table correspondente ao endereço virtual, ou cria uma nova se não existir (quando permitido).
Função mappages
A função mappages é responsável por configurar as page tables para mapear uma série de endereços virtuais para endereços físicos.

#### Processo:
Usa walk para garantir que as page tables em cada nível existam.
Preenche as entradas da page table com os endereços físicos correspondentes e define as permissões apropriadas.

#### Início da Page Table do Kernel
Na inicialização do xv6, a page table do kernel é criada e configurada pela função kvminit. Ela mapeia todas as áreas de memória física necessárias para o funcionamento do kernel, como código, dados e heap.

### 3.4 Alocação de Memória Física
#### Estrutura de Alocação de Memória
O xv6 gerencia a memória física usando uma lista ligada de páginas livres. Cada página tem 4096 bytes (4 KB) de tamanho, e o kernel usa uma estrutura simples para rastrear quais páginas estão livres ou alocadas.

Lista de Páginas Livres: Implementada como uma lista ligada, onde cada página livre aponta para a próxima página livre.

##### Cálculo de Endereços Físicos
Ao alocar ou liberar páginas, o xv6 converte endereços virtuais em endereços físicos para interagir diretamente com a memória. Por exemplo, ao liberar uma página, o endereço virtual é convertido para o endereço físico correspondente e a página é adicionada de volta à lista de páginas livres.
