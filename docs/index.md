# Oria Finance: Tokenização de RWA no Ecossistema Bitcoin

#### Projeto Final - CDE 2025 / ENE0332 - Tópicos em Engenharia (UnB)
#### Autor: Lorenzo Luiz Tavares de Souza Melo

## 1. Resumo Executivo e Contexto do Problema
<div align="justify">
A iliquidez e a opacidade são falhas estruturais crónicas nos mercados imobiliários e na gestão de propriedades municipais e estatais. Ativos de elevado valor (como empreendimentos em Smart Cities, lotes comerciais e infraestruturas) sofrem com ciclos de venda morosos, elevados custos de intermediação e barreiras de entrada que excluem o investidor de retalho.
A Oria Finance foi concebida para resolver este problema através da Tokenização de Ativos do Mundo Real (RWA - Real World Assets). Ao fracionar a propriedade em tokens digitais regulados (Security Tokens), a Oria democratiza o acesso ao investimento, reduzindo o capital mínimo necessário e garantindo uma liquidação quase instantânea em mercados secundários globais.
Originalmente desenhada para operar em redes de alto desempenho com suporte a contratos inteligentes Turing-completos (como a rede Solana), o grande desafio académico e de engenharia deste projeto é propor uma arquitetura técnica viável para replicar o ecossistema da Oria Finance de forma nativa na rede Bitcoin, respeitando as restrições arquiteturais da sua Camada 1 (L1).
</div>

## 2. A Arquitetura Original e o Caso de Uso (Oria Finance)
<div align='justify'>
No seu modelo de negócio, a Oria Finance atua como a provedora de infraestrutura tecnológica (Pilar Tecnológico) para a tokenização de propriedades, possuindo casos de uso práticos como o projeto "Smart City Cabo Verde". A plataforma separa estritamente a emissão do ativo do risco imobiliário.
Para que um RWA funcione, a rede subjacente precisa suportar três funcionalidades cruciais:
</div>

- Fracionamento e Emissão: Capacidade de cunhar (mint) milhões de frações digitais padronizadas representando um único imóvel.
- Compliance Programável (Whitelists): Como se trata de valores mobiliários (Security Tokens), as transferências secundárias só podem ocorrer entre carteiras que passaram por verificação de identidade (KYC/AML).
- Liquidez Contínua: Capacidade de transacionar estas frações 24/7 com custos residuais e liquidação instantânea.

<div align='justify'>
Numa rede como a Solana ou Ethereum (EVM), isto é resolvido facilmente através de um Contrato Inteligente (Smart Contract) global que armazena o estado de todas as contas e bloqueia transferências não autorizadas através de lógica if/else no próprio contrato.
</div>

## 3. O Desafio Arquitetural: O Protocolo Bitcoin (Layer 1)
<div align='justify'>
Transpor esta lógica diretamente para a Camada 1 do Bitcoin é impossível devido a escolhas de design fundamentais feitas por Satoshi Nakamoto para maximizar a segurança e a descentralização:
</div>

### 3.1. O Modelo UTXO vs. Modelo de Contas
<div align='justify'>
Ao contrário do Ethereum ou Solana, o Bitcoin não possui "contas" com "saldos" que um contrato possa deduzir ou incrementar. O Bitcoin utiliza o modelo UTXO (Unspent Transaction Output). Uma transação destrói inputs (UTXOs antigos) e cria novos outputs (novos UTXOs). Este modelo inviabiliza a criação de um "estado global partilhado" (como um livro de registo centralizado de todos os donos de um imóvel) na camada base, pois as transações são verificadas de forma isolada.
</div>

### 3.2. A Ausência de Turing-Completeness
<div align='justify'>
O Bitcoin Script, a linguagem de programação nativa da rede, é intencionalmente não Turing-completa. Não suporta loops nem lógicas complexas de validação de estado. Não é possível escrever um Script L1 que diga: "Permitir a transferência do Token X apenas se o endereço de destino estiver numa Whitelist gerida pela Entidade Reguladora".
</div>

### 3.3. O Problema do State Bloat (Congestionamento)
<div align='justify'>
Tentar contornar estas limitações forçando dados arbitrários para dentro da L1 não é escalável para finanças reguladas. Gravar históricos imobiliários completos, regras de conformidade e trocas secundárias na blockchain principal consumiria o espaço do bloco rapidamente, tornando as taxas proibitivas e centralizando a rede.
</div>

## 4. A Arquitetura Proposta: Validação Client-Side e Layer 2
<div align='justify'>
Para implementar a Oria Finance no Bitcoin, propomos uma mudança de paradigma: sair da "Validação em Cadeia Global" para a Validação do Lado do Cliente (Client-Side Validation - CSV), alavancando protocolos modernos como o Taproot Assets (Taro) ou o Protocolo RGB.
Nesta arquitetura, a rede Bitcoin não processa a lógica de negócio imobiliário; ela atua pura e exclusivamente como um Tribunal de Carimbos de Tempo (Timestamping) e prevenção de Gasto Duplo (Anti-Double-Spend).
</div>

### 4.1. Como a Oria Finance Operará (Passo a Passo)
#### A Camada de Âncora (Segurança Taproot):
<div align='justify'>
Utilizando a atualização Taproot do Bitcoin (BIP 341/342), os dados do contrato inteligente de um imóvel da Oria são condensados numa Árvore de Merkle (Merkle Tree). Apenas o hash da raiz (Root Hash) desta árvore é embutido na assinatura de uma transação Bitcoin normal. Para a rede principal do Bitcoin, parece apenas uma transferência padrão de Satoshis.
</div>

#### Gestão de Estado Off-Chain (Protocolo RGB / Taproot Assets):
<div align='justify'>
Toda a informação pesada — o prospeto do imóvel, as regras do regulador, a distribuição de frações — é guardada off-chain pela Oria Finance e pelos investidores envolvidos. O estado do token RWA é atrelado a um UTXO específico de Bitcoin. Quando o UTXO é gasto, a propriedade do imóvel é transferida.
</div>

#### Garantia de Compliance (KYC/AML) via PSBT:
<div align='justify'>
Para aplicar as regras regulatórias (como garantir que o comprador fez KYC), a Oria Finance pode utilizar Partially Signed Bitcoin Transactions (PSBTs) ou transações multi-assinatura (Multisig). Quando um investidor tenta enviar uma fração de imóvel, a transação exige a coassinatura do servidor de compliance da Oria. Se o destino não estiver na "Whitelist", o servidor recusa-se a assinar e o UTXO não pode ser gasto.
</div>

#### Liquidez e Negociação Secundária (Lightning Network):
<div align='justify'>
A verdadeira magia acontece na Camada 2. Uma vez que o token RWA da Oria Finance é ancorado num UTXO via Taproot Assets, esse ativo pode ser depositado num canal da Lightning Network. Na Lightning Network, as frações imobiliárias da Oria podem ser trocadas instantaneamente (em milissegundos) e com taxas quase nulas.
</div>

## 5. Conclusão e Impacto
<div align='justify'>
Migrar a infraestrutura de RWA da Oria Finance de redes baseadas em contas para o ecossistema Bitcoin é um desafio de engenharia complexo, mas altamente compensador. Exige o abandono de contratos globais em prol do modelo UTXO e da Validação Client-Side.
O resultado final, no entanto, é a criação de um mercado imobiliário tokenizado alicerçado na infraestrutura criptográfica mais segura, testada e descentralizada do mundo — a Camada 1 do Bitcoin. Ao mesmo tempo, atinge-se a velocidade e o baixo custo necessários para investidores de retalho utilizando os protocolos de Camada 2 e a Lightning Network.
</div>