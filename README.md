# writeup-chemistry-htb

OBS: CONTEM SPOILER !!!!! SE VC ESTIVER FAZENDO ESSE LAB E NAO QUISER SABER ONDE ESTAO AS FLAGS SEM NEM AO MENOS TENTAR, NAO TERMINE DE LER ESSE WRITEUP
----


alvo: 10.10.11.38

primeiro vamo começar fazendo um reconhecimento, apra procurar por portas aberta nesse ip.

![image](https://github.com/user-attachments/assets/5387a6bb-ed77-4f63-847a-272016f3d149)

vimos que tem dois serviços rodando, ssh na porta padrão e a porta 5000, vou tentar acessar essa porta 5000 na web

![image](https://github.com/user-attachments/assets/1cb50d91-f52a-4a77-9ac0-64c86b513c07)

encontramos esse CIF Analyzer, não sei o que é, então fui dar uma pesquisada, parece que é um sistema que extrai informações de ligação de arquivos de informações cristalográficas (CIF), vou clicar em register para me registrar e conheçer melhor a aplicação

![image](https://github.com/user-attachments/assets/6a239045-965c-4a11-968c-a715cf738196)

aqui vemos que da pra fazer um upload de arquivo CIF válido

cliquei em here para ele baixar esse example.cif para eu tentar entender melhor como funciona esse cif

![image](https://github.com/user-attachments/assets/efc43522-18c4-4d94-aee7-d6f6f072ece1)

analisei e mesmo assim não sai do lugar, mas vi que CIF significa Crystallographic Information Files, vou pesquisar se existe algum cve desse tipo de arquivo


![image](https://github.com/user-attachments/assets/e679f90e-0e15-4e51-9489-8289367f2264)


depois de um bom tempo de pesquisa, encontrei algo interessante no CVE-2024-23346 que se refere à uma vulnerabilidade crítica no pymatgen, uma biblioteca Python de código aberto utilizada para análise de materiais. 

https://ethicalhacking.uk/cve-2024-23346-arbitrary-code-execution-in-pymatgen-via-insecure/#gsc.tab=0

aqui encontrei o seguinte exploit:

![image](https://github.com/user-attachments/assets/87b92bb3-3eed-4849-bc76-7bae66852a49)

fui pesquisar mais sobre essa vulnerabilidade desta lib pymatgen e encontrei o seguinte exploit no github:

https://github.com/materialsproject/pymatgen/security/advisories/GHSA-vgv8-5cpj-qj2f

![image](https://github.com/user-attachments/assets/7c57f734-fabb-4950-babb-a732539724cc)

depois de um bom tempo de pesquisa, vou tentar montar meu exploit a partir do arquivo exemple.cif que encontramos lá e tentar explorar essa vuln

Quando vejo que posso fazer um upload de arquivos, logo penso em reverse shell. Fui pesquisar sobre execução de comandos de sistema em python.

Agora, voltando lá no arquivo que baixamos, contém o seguinte conteúdo:

![image](https://github.com/user-attachments/assets/4c1f7acd-890e-4ab8-85c7-722c32e5adb0)

De acordo com minha pesquisa, essa é a parte do exploit que executa comando:

_space_group_magn.transform_BNS_Pp_abc  'a,b,[d for d in ().__class__.__mro__[1].__getattribute__ ( *[().__class__.__mro__[1]]+["__sub" + "classes__"]) () if d.__name__ == "BuiltinImporter"][0].load_module ("os").system ("touch pwned");0,0,0'

vou tentar adaptar o arquivo exemple.cif e transformá-lo em um exploit.

Depois de muitas tentativas e erros, finalmente consegui montar meu exploit. Ficou da seguinte forma:

![image](https://github.com/user-attachments/assets/2ecac304-ac7f-42bf-bfde-a2e8e7a9e7ed)

o intuito desse exploit eh ver se conseguimos um RCE (execução de código remoto) no servidor que processa arquivos .cif. O exploit injeta um comando de shell reverso que tenta se conectar de volta ao servidor do atacante (eu), fornecendo uma shell remota.

Deixei meu servidor escutando na porta que coloquei no exploit:

![image](https://github.com/user-attachments/assets/f0a62b3a-b464-4d44-9ee5-ffec766eb42d)

agora, vamos fazer o upload do meu arquivo exemple.cif (exploit).


![image](https://github.com/user-attachments/assets/0d08ec55-1fe4-474a-80e2-94e8fa7242fc)

arquivo upado, agora, precisamos executar ele. Para isso, irei clicar bo botao view para abrir(executar) o arquivo, e ver se tivemos respostas.

nao deu certo, mas depois de alguns ajustes consegui.

![image](https://github.com/user-attachments/assets/03d862d3-0e99-4d58-b515-02ab5cb6289e)

conseguimos executar comandos no sistema, encontramos uma vulnerabilidade RCE, estamos conectados no alvo atravez de um reverse shell. Agora vamo explorar diretorios e arquivos do servidor.

![image](https://github.com/user-attachments/assets/744893cc-a2b1-4c8b-9df7-1846ac064779)


apos explorar os diretorios, o unico arquivo suspeito que encontrei foi um database.db

![image](https://github.com/user-attachments/assets/1d22215b-88e0-406a-b7e3-46745f19eb04)

enviei esse arquivo para meu pc, para analisar.

acessei o db usando o sqlite3 e encontramos o seguinte:

![image](https://github.com/user-attachments/assets/515a2d39-9fc8-444c-b331-c4d252febfda)

users e passwords criptografados, provavelmente uma hash md5, vamos descobrir se nosso velho amigo crackstation.net da conta dessas.

apos algumas tentativas, so consegui quebrar esse hash que corresponde ao user "rosa":

![image](https://github.com/user-attachments/assets/64e4bf5e-cd0a-4aa8-b015-669e19dce193)

vamos testar o login na aplicacao com esse user e senha que descobrimos

![image](https://github.com/user-attachments/assets/20cbf01c-e46a-4133-beca-13caa9306fbe)

logamos.
![image](https://github.com/user-attachments/assets/5a00ef04-2325-4261-95ef-3513af3e5696)

sabe mais onde podemos tentar logar com esse user? Exatamente.

Lembra quando usamos o nmap para procurar por portas e servicos abertos? A porta ssh esta open.

![image](https://github.com/user-attachments/assets/35e5715e-4419-4c32-8e2f-007f94d394ed)

Dei enter e logo de cara veio aquela pergunta de yes or no, entao, esse user existe. Agora vamos colocar a hash que quebramos.

![image](https://github.com/user-attachments/assets/0ff93f72-08bd-4c46-8b98-ab7da1c5fd60)

Estamos dentro. Conseguimos acessar o servidor da aplicacao via ssh.

Agora eh continuar explorando, logo de cara, dei um ls para ver oq tinha nesse dir e finalmente... Encontramos a primeira flag:

![image](https://github.com/user-attachments/assets/996d09fe-4a94-45fa-b9ed-597b33023009)

Precisamos encontrar agora a flag root.

Depois de muito tempo procurando e nao encontrar nada, resolvi usar o nmap mais uma vez e encontrei algumas portas abertas localmente. Entao fiz um tunelamento ssh para permitir meu acesso `a um servico que esta disponivel apenas localmente no host como se ele estivesse rodando em minha maquina:

![image](https://github.com/user-attachments/assets/452bea69-1b8a-44c0-a60a-74e5bcba6226)

Acessei na web e fui analisar.

era uma outra aplicacao rodando, que coletava dados e exibia em graficos. Entao iniciei minha pesquisa nessa aplicacao usando ferramentas e scripts de coleta de dados. Depois de ter pesquisado batante, usei o whatweb para analistar as tecnologias utilizadas.

![image](https://github.com/user-attachments/assets/d051fdaf-fc97-4503-88ed-da17a684107a)

Encontramos tecnologias utilizadas pelo servidor e suas versões. Vimos que o servidor http é python e a biblioteca é aiohttp/3.9.1. Então fui pedir ajuda ao pai dos burros (Google) para ver se tinha algum cve dessa lib do python.

![image](https://github.com/user-attachments/assets/583a7dff-e576-4b4e-a5e8-d26b33c5d4f5)

Encontrei o seguinte cve:

https://github.com/z3rObyte/CVE-2024-23334-PoC

peguei o exploit desse cve, modifiquei de acordo com o meu cenário e executei.

![image](https://github.com/user-attachments/assets/c749bcaa-36d4-448e-a1e2-21c132fbe039)

consegui burlar o sistema e finalmente obtive a flag root.

