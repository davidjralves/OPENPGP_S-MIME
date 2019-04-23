# EFAIL-A falha de segurança no OpenPGP e S/MIME
 Actualmente nas comunicações de _e-mail_, as mensagens são encriptadas usando criptografia simétrica, sendo as chaves geradas apenas 
para uma comunicação e são baseadas num segredo partilhado que é negociado no início de cada  comunicação, e por isso usa-se o protocolo
TLS na comunicação  de _e-mail standard_. Contudo  por vezes existem  situações em que as  comunicações necessitam de ser altamente 
confidenciais e o protocolo TLS pode não ser suficiente, (até porque o protocolo TLS  deriva do SSL, um protocolo que acabou por 
ser  proibído  devido  a  várias  vulnerabilidades  de  segurança,  sobretudo  a  ataques _man-in-the-middle_). Para responder a estas
necessidades e vulnerabilidades do TLS, implementaram-se  protocolos  como  o OpenPGP e S/MIME que têm uma encriptação 
_end-to-end_, (sistema de comunicações no  qual  apenas  os  intervenientes  da comunicação podem ter acesso às mensagens, 
não permitindo sequer o ISP(fornecedor  de  _internet_)  ter  acesso  às  mesmas). Como nota histórica, Edward Snowden, 
quando contactou a jornalista Laura Poitras sobre  aquilo  que  viria  a  ser  o  grande escândalo da falta de privacidade na internet,
usou OpenPGP.


Em 2018, 8 investigadores, que cujo nome deve ser referido, Damian Poddebniak, Christian Dresen, Jens Müller, Fabian Ising, Sebastian 
Schinzel, Simon Friedberger, Juraj Somorovsky e Jörg Schwenk, como parte da sua participação na USENIX, descobriram vulnerabilidades 
nos protocolos OpenPGP e S/MIME em comunicações de _e-mail_, o que criou uma grande surpresa na comunidade criptográfica e dividiu 
muitas opiniões, havendo desde aqueles que diziam que  o OpenPGP estava “morto” e não devia ser usado, e aqueles que diziam que o 
problema não estava no OpenPGP, mas sim em alguns clientes de _e-mail_ que usam o protocolo. A esta falha foi-lhe atribuído o nome de EFAIL, e basicamente retrata o seguinte ataque: numa primeira fase o atacante tem que ter acesso aos _e-mails_ encriptados, (isto pode ser feito através de captura de trafego de uma rede ou atacando contas de _e-mail_, servidores ou computadores, entre outros). O atacante altera o _e-mail_ encriptado, adicionando, (usando HTML), conteúdo externo, como por exemplo, imagens colocadas num servidor com um determinado URL, e envia-o para a vítima. O cliente de _e-mail_ da vítima desencripta o _e-mail_ e lê o conteúdo externo de um dado URL, isto vai fazer com que em alguns clientes de _e-mail_, o texto limpo seja possível de obter pelo atacante. 


Existem dois tipos de ataques EFAIL: _direct exfiltration_ e _CBC/CFB Gadget Attack_.

O _direct exfiltration_ funciona da seguinte forma: O atacante cria um novo código HTML, constituído por três partes, a primeira 
contém o URL de uma imagem, por exemplo, deixando aberto o atributo _src_ o que faz com que se possa adicionar mais dados a esse 
atributo, e é isso que é feito na segunda parte, esta contém o _e-mail_ encriptado que é assim adicionado ao atribuito _src_ que já 
continha uma imagem, a terceira parte serve apenas para fechar o conteúdo do atributo src. De seguida, o atacante envia este _e-mail_ 
para a vítima. O cliente de _e-mail_ da vítima desencripta a segunda parte do HTML enviado e coloca o conteúdo desencriptado das três 
partes num único HTML. De seguida o cliente de _e-mail_, vai fazer um pedido ao URL presente na primeira parte do HTML 
desencriptado, codificando todos os  caracteres não imprimíveis, (por exemplo %20 para espaços entre palavras), adiciona ao URL o 
conteúdo da mensagem  desencriptada. Como é óbvio, uma vez que o atacante é administrador do website onde se encontra a imagem e por 
isso poderá ver facilmente quais foram os pedidos feitos ao seu servidor do _website_, podendo assim obter o texto limpo.


O _CBC/CFB Gadget Attack_ é um método, na minha opinião mais lento e complicado, no entanto também pode ser eficaz e deve-se ao facto de 
a cifra utilizada no protocolo S/MIME ser CBC, no qual, um atacante pode alterar os blocos de texto limpo, se souber qual o 
texto limpo, o que torna o S/MIME ainda mais vulnerável, uma vez que os _e-mails_ encriptados usando S/MIME geralmente começam 
por “_Content-type: multipart/signed_". Sabendo isto poderá assim tentar-se combinar vários blocos de texto limpo que já conhecemos o 
seu conteúdo, juntamente com blocos de texto cifrado que não sabemos o conteúdo e de seguida desencriptar, como consequência disto, o 
texto limpo fica ilegível, o atacante poderá corrigir isto com a adição de gadgets, (técnicas de alterar a saída de determinadas 
funções), sendo que assim o atacante poderá inserir no meio dos blocos de texto limpo que não conhece, as tags de HTML que inserem a 
leitura de conteúdos externos ao _e-mail_. Este método funciona também no OpenPGP, que usa a cifra CFB. No entanto ao contrário do 
S/MIME, o OpenPGP disponibiliza _Modification Detection Code (MDC)_ que consegue detetar texto limpo modificado, no entanto  estes 
investigadores do EFAIL descobriram que alguns clientes de _e-mail_ simplesmente mostram um aviso quando existe um texto limpo 
modificado e continuam a mostrar o seu conteúdo.


Estes investigadores deixaram algumas recomendações aos fornecedores de clientes de _e-mail_ para evitar estes ataques, algumas delas são: Remover a desencriptação no cliente de _e-mail_, sendo que para isso as chaves privadas de remetente e do destinatário não poderiam estar armazenadas no cliente de _e-mail_, e sendo assim  o cliente de _e-mail_ apenas serveria para enviar e receber mensagens encriptadas, e depois essas mensagens encriptadas seriam desencriptadas numa aplicação à parte, desta forma o atacante não poderia inserir a abertura de conteúdo externo; Desativar o conteúdo ativo presente em HTML ou JavaScript, tais como a apresentação de imagens, entre outros;Atualizar os standards do S/MIME e do OpenPGP.


O facto do EFAIL ter gerado polémica deve-se também por ter mostrado estas falhas em clientes de _e-mail_ usados por um grande 
número de pessoas. Alguns do clientes de _e-mail_ nos quais se verificaram os ataques EFAIL foram: Thunderbird, Apple Mail, iOS Mail, GMail, Outlook 2016, Windows 10 Mail, entre outros. De todos, aqueles dos quais não se conseguiu obter com sucesso os ataques EFAIL tanto em S/MIME como em OpenPGP, são o Claws e o Mutt (curiosamente, não tão populares como aqueles nos quais se conseguiu atacar).

Por tudo isto, e analisados todos os aspetos, na minha opinião, não creio que o OpenPGP ou S/MIME estejam “mortos“, como dizem 
alguns investigadores, sobretudo porque aquilo que se provou foi que o problema não está no OpenPGP ou no S/MIME mas sim nos 
clientes de  _e-mail_. Eu, se tivesse que escolher entre OpenPGP ou S/MIME, escolhia OpenPGP para as minhas comunicações sobretudo 
pelo facto de disponibilizar _Modification Detection Code (MDC)_, o que mesmo que em alguns clientes de _e-mail_ apenas seja mostrado um aviso, já é um ganho significativo em relação ao S/MIME. No entanto também acho que os mecanismos do _OpenPGP_ e do S/MIME(sendo que na minha opinião o S/MIME cairá em desuso) deviam ser atualizados, para dificultar estes ataques EFAIL.

David Ressurreição Alves

---
Fontes:

https://efail.de/

https://en.wikipedia.org/wiki/Transport_Layer_Security

https://en.wikipedia.org/wiki/Pretty_Good_Privacy

https://en.wikipedia.org/wiki/End-to-end_encryption

https://en.wikipedia.org/wiki/S/MIME

---
Como forma de demonstração deste  tipo de ataque, fiz um conjunto de scripts em python, e para se poder observarem devem-se seguir os seguintes passos:

1.Executar _script_ Alice.py

2.Escrever nesse programa a mensagem a enviar para o Bob

3.Executar a _script_ EFAIL.py

4.Nesse programa, inserir o endereço para o qual as mensagens vão ser redirecionadas (No meu exemplo eu escolhi efail.pt)

(Verificar que o site digitado é adicionado ao lista.txt)

5.Executar a _script_ cliente_de_email_Bob.py 

6.Verificar que o Bob recebeu a mensagem da Alice 

7.Abrir o ficheiro criado para o endereço digitado no passo 4, e verificar que sao mostradas as mensagens (url+mensagem)
