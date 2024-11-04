# whiteup-chemistry-htb


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

o intuito desse exploit eh ver se conseguimos um RCE (execução de código remoto) no servidor que processa arquivos .cif. O exploit injeta um comando de shell reverso que tenta se conectar de volta ao servidor do atacante (eu), fornecendo uma shell remota









