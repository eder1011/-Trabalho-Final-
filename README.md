
***Este código realiza a integração do Google Drive com o ambiente do Google Colab de forma controlada e segura.
Inicialmente, são importados os módulos drive, responsável pela montagem do Google Drive, e os, utilizado para verificar diretórios e executar comandos do sistema.
Em seguida, o código verifica se o diretório /content/drive já existe; caso exista, tenta desmontar uma montagem anterior do Google Drive,
ignorando erros caso o Drive não esteja montado. Após isso, qualquer arquivo residual presente no diretório é removido, garantindo um ambiente limpo.
O código então cria o diretório de montagem, se necessário, ou esvazia seu conteúdo caso ele já exista. Por fim, o Google Drive é montado novamente em /content/drive
com a opção force_remount=True, assegurando uma nova montagem sem conflitos com sessões anteriores.***


```Python

from google.colab import drive
import os

# Desmonta o drive se já estiver montado e limpa o diretório
if os.path.isdir('/content/drive'):
    try:
        drive.flush_and_unmount()
    except ValueError:
        pass # Drive not mounted, ignore error
    os.system('rm -rf /content/drive/*') # Remove any residual files

# Cria o diretório de montagem se não existir e o esvazia
if not os.path.exists('/content/drive'):
    os.makedirs('/content/drive')
else:
    os.system('rm -rf /content/drive/*')

drive.mount('/content/drive', force_remount=True)
```

***O comando apt-get update atualiza a lista de pacotes disponíveis no sistema. Em seguida, são instaladas as ferramentas BWA (alinhamento de sequências), SAMtools (manipulação de arquivos SAM/BAM) e BEDtools (operações com arquivos genômicos em formato BED). Após isso, uma mensagem indicativa é exibida e a lista de pacotes é atualizada novamente para garantir versões consistentes. Por fim, são instaladas as ferramentas complementares BCFtools e VCFtools, utilizadas para manipulação, filtragem e análise de variantes genéticas em arquivos VCF/BCF.***

```bash

apt-get update

apt-get install -y bwa
apt-get install -y samtools
apt-get install -y bedtools

echo #Instalando ferramentas complementares..."
apt-get update

apt-get install -y bcftools
apt-get install -y vcftools
```

***

Realiza o download e a instalação local do GATK (Genome Analysis Toolkit) no ambiente de execução. O comando wget é utilizado para baixar silenciosamente (-q) o arquivo compactado da versão 4.1.8.1 diretamente do repositório oficial do Broad Institute no GitHub. Em seguida, o comando unzip extrai o conteúdo do arquivo ZIP, disponibilizando o executável e seus componentes no diretório de trabalho. Por fim, o arquivo compactado é removido com rm para liberar espaço em disco, mantendo apenas os arquivos necessários para o uso do GATK.***

```bash

wget -q https://github.com/broadinstitute/gatk/releases/download/4.1.8.1/gatk-4.1.8.1.zip
unzip -q gatk-4.1.8.1.zip
rm gatk-4.1.8.1.zip
```

***Este comando realiza o download do Picard Tools diretamente do repositório oficial do Broad Institute no GitHub. O wget é utilizado para obter o arquivo executável picard.jar, que contém um conjunto de ferramentas amplamente usadas para manipulação e análise de arquivos BAM, SAM e métricas de sequenciamento. Após o download, o arquivo pode ser executado via Java para tarefas como marcação de duplicatas, validação de arquivos e coleta de estatísticas de qualidade.***

```bash
wget https://github.com/broadinstitute/picard/releases/download/2.24.2/picard.jar
```

***Define o diretório principal do projeto no Google Drive e cria a estrutura de pastas para armazenar genomas de referência (hg19 e hg38) e arquivos FASTQ. O uso da opção -p garante que as pastas sejam criadas sem erro caso já existam, facilitando a organização e reprodutibilidade do pipeline.***

```bash

MeuDrive="/content/drive/MyDrive/TRABALHO_FINAL"

mkdir -p $MeuDrive/referencia/hg38
mkdir -p $MeuDrive/referencia/hg19
mkdir -p $MeuDrive/dados/fastq
```

***Realiza o download do arquivo compactado do cromossomo 10 do genoma humano hg19 a partir do UCSC Genome Browser, descompacta o conteúdo em tempo real e salva o arquivo FASTA no diretório de referência do projeto. Essa sequência é utilizada como genoma de referência nas etapas subsequentes de alinhamento e análise.***

```bash

MeuDrive="/content/drive/MyDrive/TRABALHO_FINAL"


curl -s "https://hgdownload.soe.ucsc.edu/goldenPath/hg19/chromosomes/chr10.fa.gz" | \
   gunzip -c > "$MeuDrive/referencia/hg19/hg19.fasta"

```

***Exibe as 10 primeiras linhas do arquivo FASTA do genoma de referência hg19, permitindo verificar se o download e a descompactação foram realizados corretamente, bem como confirmar o formato do arquivo antes de sua utilização no pipeline.***

```bash

MeuDrive="/content/drive/MyDrive/TRABALHO_FINAL"

head -n 10 $MeuDrive/referencia/hg19/hg19.fasta

```

***Output:***

```
>chr10
NNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNN
NNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNN
NNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNN
NNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNN
NNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNN
NNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNN
NNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNN
NNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNN
NNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNN
```

***Cria os arquivos (5 arquivos auxiliares .amb, .ann, .bwt, .pac, .sa) de índice do BWA a partir do genoma de referência hg19 (cromossomo 10). A opção -a bwtsw define o algoritmo de indexação recomendado para sequências genômicas longas (>2GB), permitindo o alinhamento eficiente das leituras de sequenciamento.***

```bash

MeuDrive="/content/drive/MyDrive/TRABALHO_FINAL"

bwa index \
  -a bwtsw \
  $MeuDrive/referencia/hg19/hg19.fasta

```

***Gera o índice FASTA (.fai) do genoma de referência hg19 utilizando o SAMtools. Esse índice permite acesso rápido a regiões específicas do genoma, sendo essencial para etapas posteriores de alinhamento, visualização e análise dos dados de sequenciamento (visualizadores como IVG).***

```bash

MeuDrive="/content/drive/MyDrive/TRABALHO_FINAL"

samtools faidx $MeuDrive/referencia/hg19/hg19.fasta

```

***Remove versões anteriores do arquivo de dicionário, caso existam, e cria o arquivo .dict do genoma de referência hg19 utilizando o Picard. Esse dicionário contém informações estruturais das sequências (nomes, tamanhos e ordem dos cromossomos) e é obrigatório para ferramentas como o GATK.***


```bash
MeuDrive="/content/drive/MyDrive/TRABALHO_FINAL"

rm -f $MeuDrive/referencia/hg19/hg19.dict

java -jar picard.jar CreateSequenceDictionary \
REFERENCE=$MeuDrive/referencia/hg19/hg19.fasta \
OUTPUT=$MeuDrive/referencia/hg19/hg19.dict
```

***Este script realiza uma verificação automática da preparação do genoma de referência hg19, conferindo a presença de todos os arquivos essenciais gerados nas etapas anteriores: arquivo FASTA, índice do SAMtools (.fai), dicionário do Picard (.dict) e os arquivos de índice do BWA. Ao final, é exibido um resumo do status da preparação, indicando se o genoma está completo e pronto para ser utilizado nas análises de alinhamento e chamada de variantes.***

```bash
MeuDrive="/content/drive/MyDrive/TRABALHO_FINAL"

echo "🔍 Verificação final da preparação do genoma:"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

arquivos_essenciais=(
    "hg19.fasta:Genoma FASTA"
    "hg19.fasta.fai:Índice samtools"
    "hg19.dict:Dicionário Picard"
    "hg19.fasta.amb:BWA .amb"
    "hg19.fasta.ann:BWA .ann"
    "hg19.fasta.bwt:BWA .bwt"
    "hg19.fasta.pac:BWA .pac"
    "hg19.fasta.sa:BWA .sa"
)

total=0
presentes=0

for item in "${arquivos_essenciais[@]}"; do
    arquivo=$(echo $item | cut -d: -f1)
    descricao=$(echo $item | cut -d: -f2)

    if [ -f "$MeuDrive/referencia/hg19/$arquivo" ]; then
        tamanho=$(du -h "$MeuDrive/referencia/hg19/$arquivo" | cut -f1)
        echo "✅ $descricao ($tamanho)"
        ((presentes++))
    else
        echo "❌ $descricao - AUSENTE"
    fi
    ((total++))
done

echo ""
echo "📊 RESUMO: $presentes/$total arquivos presentes"

if [ $presentes -eq $total ]; then
    echo "🎉 PREPARAÇÃO COMPLETA! Genoma pronto para uso."
else
    echo "⚠️ Alguns arquivos estão faltando. Revise as etapas anteriores."
fi

echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

```

***Output:***

```


🔍 Verificação final da preparação do genoma:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Genoma FASTA (132M)
✅ Índice samtools (512)
✅ Dicionário Picard (512)
✅ BWA .amb (512)
✅ BWA .ann (512)
✅ BWA .bwt (130M)
✅ BWA .pac (33M)
✅ BWA .sa (65M)

📊 RESUMO: 8/8 arquivos presentes
🎉 PREPARAÇÃO COMPLETA! Genoma pronto para uso.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

***Executa um teste funcional da indexação do genoma hg19, extraindo uma região específica do cromossomo 10 utilizando o samtools faidx. A exibição correta da sequência confirma que o arquivo FASTA e seu índice foram gerados adequadamente e que o genoma está pronto para uso nas etapas de alinhamento e análise do pipeline.***

```bash
MeuDrive="/content/drive/MyDrive/TRABALHO_FINAL"

echo "🧪 Teste: Extraindo região chr10:1000-1100"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

samtools faidx "$MeuDrive/referencia/hg19/hg19.fasta" chr10:57227864-57227930

echo ""
echo "✅ Se você viu a sequência acima, a indexação funcionou perfeitamente!"
echo "📏 Essa região tem exatamente 101 bases (1100-1000+1)"
```

***Output:***

```
🧪 Teste: Extraindo região chr10:1000-1100
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
>chr10:57227864-57227930
GTGTTAGTAAACAAGCAGTTTCTCAAGAGCAGGGGGGAAAAGTTAGTGACAGAAATATGT
TCAAACA

✅ Se você viu a sequência acima, a indexação funcionou perfeitamente!
📏 Essa região tem exatamente 101 bases (1100-1000+1)
```

***Este script verifica a existência do diretório de dados FASTQ e lista os arquivos de sequenciamento presentes. Caso nenhum arquivo seja encontrado, mensagens informativas são exibidas para auxiliar na identificação de problemas de organização ou cópia dos dados. Essa etapa garante que os dados de entrada necessários para o alinhamento estejam disponíveis antes da execução do pipeline.***

```bash
MeuDrive="/content/drive/MyDrive/TRABALHO_FINAL"

echo "📁 Verificando arquivos FASTQ..."
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

if [ -d "$MeuDrive/dados/fastq" ]; then
    echo "📄 Arquivos encontrados:"
    ls -lh "$MeuDrive/dados/fastq/"*.fastq 2>/dev/null || {
        echo "❌ Nenhum arquivo FASTQ encontrado!"
        echo "📝 Verifique se os arquivos foram copiados corretamente."
    }
else
    echo "❌ Diretório dados/fastq não encontrado!"
    echo "📝 Crie a estrutura de diretórios e copie os arquivos FASTQ."
fi

echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

```
***Output:***

```
📁 Verificando arquivos FASTQ...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📄 Arquivos encontrados:
-rw------- 1 root root 66M Jan  7 18:31 /content/drive/MyDrive/TRABALHO_FINAL/dados/fastq/cap-ngse-b-2019-chr10_S1_L001_R1_001.fastq
-rw------- 1 root root 68M Jan  7 18:30 /content/drive/MyDrive/TRABALHO_FINAL/dados/fastq/cap-ngse-b-2019-chr10_S1_L001_R2_001.fastq
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

***Este script localiza um arquivo FASTQ de leituras reversas (R2), exibe exemplos iniciais das sequências e calcula estatísticas básicas, como número total de reads e tamanho do arquivo. Essa etapa permite verificar a integridade e o formato dos dados de sequenciamento, assegurando que os arquivos de entrada estão adequados para as etapas de alinhamento.***

```bash
MeuDrive="/content/drive/MyDrive/TRABALHO_FINAL"

echo "📄 Analisando arquivo FASTQ R2..."
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

arquivo_r2=$(find "$MeuDrive/dados/fastq/" -name "*R2*.fastq" | head -1)

if [ -f "$arquivo_r2" ]; then
    echo "📁 Arquivo: $(basename "$arquivo_r2")"
    echo ""
    echo "🔍 Primeiras 12 linhas (3 reads completos):"
    head -12 "$arquivo_r2"

    echo ""
    echo "📊 Estatísticas do arquivo:"
    total_linhas=$(wc -l < "$arquivo_r2")
    total_reads=$((total_linhas / 4))
    echo "• Total de linhas: $(printf "%'d" $total_linhas)"
    echo "• Total de reads: $(printf "%'d" $total_reads)"
    echo "• Tamanho do arquivo: $(du -h "$arquivo_r2" | cut -f1)"
else
    echo "❌ Arquivo FASTQ R2 não encontrado!"
    echo "📝 Verifique se os arquivos estão na pasta correta."
fi

echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

```

***Output:***

```
📄 Analisando arquivo FASTQ R2...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📁 Arquivo: cap-ngse-b-2019-chr10_S1_L001_R2_001.fastq

🔍 Primeiras 12 linhas (3 reads completos):
6��"�6Yѓ���ۗxO���p�����U�1�w;֦+��y*���j��HS���&�9�q���`���	�>�k:�?�^���#b�V�'��-�4-a��z@Ƒ�{�~]�{w��C��oL�(l@�(D��0Lwf87����JU,R^����t|�!\��\u��w�@Oz_���s�_�6��od}6#
�51��ǁ&�4��i����{�|=��*� T����P��\�U��hL8-OW0!0��Eo*<����f���@��xe��UK_�a2>�K�T�Ւ8�ݓ�Ɖ�L}%�X,ڗO<�`�<u�G�j�^�_����X�i��������>%�H���B�ᚇ����tuY4|��'���c���uW3���m�:�O"&��a��E��ŭ��y(���.��R������(����Z��V��~���
b��+t_�E��ݴ/b�b�>���ɼ�CѱlSud_�Ӱ�4ч'�Jx6��P�iYޔlB^�ފϜ��/�u��t��l?`޴����t�y�K�����<Pi������*'��*W�F�ѕ Pw�:���������<0YG������@T����l�:�h�	��˸�g,aSǷ��|_v�
|��˿byn��1YZ�OמO�2�&h�k`
��-���ڲ�M}�;��n=���냠�2���2���^*�`�J����VYG��)��Z���BByS������Q�f�G�"�̋<���<p�S�i�0WO�&����>PV�������Z���MDj�zL��Kڡ���{;߿.����#��*��p���+�Y�[	0�����+� ��FK��
B�1�=�����#
<8cM�!3v�<f\�	���M)������F�7&��k�~5���m�������
Q:�,�9�E�9�ʢ�F�6�o�����:�8f{�ƕ\�2��.�hM�[���:WM�G�y��L���L'$d]@]с��
##U(�+��'�*L�X:B�&�%��U��(1���,�	Tf�&6E�]����!p(�1/X�͠i��?r<�b���۶K�]`��
�-`��zD`Q�����Z��B�G��Y�4y-"5y�z�/���]�cV=�NN��"Ngd��ٟKX�Iۊ�E"KjW�uX�w��Ŭ�����v�1���$dx����y�$���es)����hJ�[s҈�.�Ъ�(�� Ȏ�8Lָ��x��ͥP�d��Tx�#V�w���e��+;y����0S��c���c�c��-���5�
�rY�@�;o)�mK#���0<Og���FJ႔Q�u#��U��v��ÂC�.���p`�r�s]S6c�e���F�e�`����:Iv�_H�z�?��`��zF'�9�E������Ŭ�t�J$܊�0.AE3���פx���p�LP���o3A"K��@\L�1)����|����:��e	ph8�o�>o���Y5Z}Gb%��w�W\����,��pB6jB9V3C|�B"�Kk$��^6F]���7��10�DЕ�m��4oXK��,�r�
GޗԄ��Z~��r�N矎|��##���i<����K!0��Wf��D����GA+>ը��B��8��E~��6�^�:�>�1����1M����s���Z�[�=l#�WAQBp�A�z�DeG��[Gc�� \W���J"����R��4�h��TȻ'E�EP�W]5f��j��	��l���Ѽ���ٙ�X.f�S���͂5J�BO!i.�*�^��y���J����y$A@����l+l����|ͅ�A�7Y��̺�A��jhHT�G��I̳�lX�%�L�u�� �C�qi:�L0O������!��}M�̽>q����N˃��� X`�eT��,1���Ű�Z���ܹR�*�5��c���p�,������4�8��׾M���f�Ʊ��@��d���ro�e/���s/����U�B�Հ������L,�49EtV�Q�d$����W���z��ʍi�Gqg8a�y����zYWu�h�X�!)�L��e;�o\tF�T��EMDwμ�f쑍H�aRp����y;�{~ܵ�)=e��O\��П�\���Z5J<U��)3�H�`��D>�����V�J�aE�-�*�~~q}��y

📊 Estatísticas do arquivo:
• Total de linhas: 273,651
• Total de reads: 68,412
• Tamanho do arquivo: 68M
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

***Conta o número total de linhas do arquivo FASTQ de leituras diretas (R1). Como cada read FASTQ é composta por quatro linhas, esse valor permite estimar o número total de leituras de sequenciamento, auxiliando na verificação da integridade e do volume dos dados de entrada.***

```bash
wc -l /content/drive/MyDrive/TRABALHO_FINAL/dados/fastq/cap-ngse-b-2019-chr10_S1_L001_R1_001.fastq

```

***Output:***

```
272730 /content/drive/MyDrive/TRABALHO_FINAL/dados/fastq/cap-ngse-b-2019-chr10_S1_L001_R1_001.fastq
```

***Cria o diretório de saída para arquivos BAM e executa o alinhamento das leituras pareadas (R1 e R2) contra o genoma de referência hg19 utilizando o BWA-MEM. São definidos metadados do grupo de leitura (Read Group), como identificador da amostra, biblioteca e plataforma de sequenciamento, garantindo compatibilidade com ferramentas posteriores (ex.: GATK). O resultado do alinhamento é salvo no formato SAM, contendo as leituras alinhadas ao genoma de referência.***

```bash
MeuDrive="/content/drive/MyDrive/TRABALHO_FINAL"

mkdir -p $MeuDrive/dados/bam

SAMPLE="cap-ngse-b-2019"
Biblioteca="Exoma"
Plataforma="Illumina"

arquivo_r1="$MeuDrive/dados/fastq/cap-ngse-b-2019-chr10_S1_L001_R1_001.fastq"
arquivo_r2="$MeuDrive/dados/fastq/cap-ngse-b-2019-chr10_S1_L001_R2_001.fastq"

bwa mem -K 100000000 \
    -R "@RG\tID:$SAMPLE\tSM:$SAMPLE\tLB:$Biblioteca\tPL:$Plataforma" \
    "$MeuDrive/referencia/hg19/hg19.fasta" \
    "$arquivo_r1" \
    "$arquivo_r2" > "$MeuDrive/dados/bam/$SAMPLE.sam"

```

***Converte o arquivo SAM gerado no alinhamento para o formato BAM, realiza a ordenação das leituras por coordenada genômica e cria o arquivo de índice (.bai) utilizando o SAMtools. Essas etapas são essenciais para otimizar o desempenho e permitir o uso do arquivo BAM em análises subsequentes, como visualização, chamadas de variantes e processamento com o GATK.***

```bash
MeuDrive="/content/drive/MyDrive/TRABALHO_FINAL"
SAMPLE="cap-ngse-b-2019"

samtools sort -O bam -o "$MeuDrive/dados/bam/$SAMPLE.sorted.bam" "$MeuDrive/dados/bam/$SAMPLE.sam"
samtools index "$MeuDrive/dados/bam/$SAMPLE.sorted.bam"
```

***Exibe as cinco primeiras linhas do arquivo SAM gerado no alinhamento, permitindo verificar o cabeçalho e o formato dos registros, incluindo informações de referência e grupos de leitura. Essa inspeção inicial confirma que o alinhamento foi executado corretamente antes das etapas de processamento do BAM.***

```bash
MeuDrive="/content/drive/MyDrive/TRABALHO_FINAL"
SAMPLE="cap-ngse-b-2019"

head -5 "$MeuDrive/dados/bam/$SAMPLE.sam"
```

***Output:***

```
@SQ	SN:chr10	LN:135534747
@RG	ID:cap-ngse-b-2019	SM:cap-ngse-b-2019	LB:Exoma	PL:Illumina
@PG	ID:bwa	PN:bwa	VN:0.7.17-r1188	CL:bwa mem -K 100000000 -R @RG\tID:cap-ngse-b-2019\tSM:cap-ngse-b-2019\tLB:Exoma\tPL:Illumina /content/drive/MyDrive/TRABALHO_FINAL/referencia/hg19/hg19.fasta /content/drive/MyDrive/TRABALHO_FINAL/dados/fastq/cap-ngse-b-2019-chr10_S1_L001_R1_001.fastq /content/drive/MyDrive/TRABALHO_FINAL/dados/fastq/cap-ngse-b-2019-chr10_S1_L001_R2_001.fastq
NB551003:113:HG73YBGX7:1:11101:1071:13957	99	chr10	99237520	60	151M	=	99237651	282	GAACTTAGAAGGCAATAATTTTTGCTATTGAATCCCAGTTATGTCAAGGGGTAGAGACAGAGGAGAATACCAACATGACTGTTATCCCTCACCTGTACATGCACTTCCTGGAAGATGGCTTTAAGCACAGAAACAGCCAGCCCTGGGGGCA	AAAAAEAEEEEEEEEE6EEEEEEEEEEEEEEEEEEAEEEEEEEEEAEEEEEEEE/EEEEEEEEEEEEEEEEE/</EEE/EE<EEEEE/EE<A6EEEEAEE/EEAEEEEEEEE/EAEEE<EEEEEEA/EEEE/EAEE/<6AA<<A<A/A<</	NM:i:0	MD:Z:151	MC:Z:151M	AS:i:151	XS:i:0	RG:Z:cap-ngse-b-2019
NB551003:113:HG73YBGX7:1:11101:1071:13957	147	chr10	99237651	60	151M	=	99237520	-282	AACAGCCAGCCCTGGGGGCAGGGCCACACACAGGCTCTGGGGGAGAGGAGAAGGTACGTGAATACCGAAGGAATTGCAGCANNNNCTCCAAGACACAACCCTACGACAGGCCTAATTAGCTACTGTAAGAATCACAGCATCCTGGTTGAGG	EAEEEAEEEEEEEA<A<A<EA6AAAEAAAA<A<<EE/<66/EEAEEEEEEA<EE/EE/EEEEEEEEEEE/EEEE/EEE<EE####EEEEEEEEEEAEEEEEEEEEEEEAEEEEEEEEEEEEEEEEEEEAEEEEEEE/EEEEEEEEEAAAAA	NM:i:4	MD:Z:81T0G0G0C66	MC:Z:151M	AS:i:143	XS:i:0	RG:Z:cap-ngse-b-2019

```

***Tenta exibir as primeiras linhas do arquivo BAM ordenado, o que resulta em saída ilegível por se tratar de um arquivo binário. Este comando é usado apenas como uma verificação rápida da existência do arquivo. Para inspeção adequada do conteúdo, devem ser utilizados comandos como samtools view ou samtools view -H.***

```bash
MeuDrive="/content/drive/MyDrive/TRABALHO_FINAL"
SAMPLE="cap-ngse-b-2019"

head -2 "$MeuDrive/dados/bam/$SAMPLE.sorted.bam"
```

***Output:***
```
�BCe���N�0ǫ�r텗}�݇l��9�C0�"!��d��׳�6>��Hf�����9�������3?�8����ңD��ႦR�Y^0�5;���G�MIL�z�t��n��Y�L�lLS���u�q�;&�p~�rG��U
}�ζ�'�L���<ya�������[:�pI�`��:��!b~�d�H�G�X=R�|���##���2�2���m�ć�9�t��<�5J��%/Ҝ�56��b��) c����bgt�S�Įi�8 �i���xgo���PRn�����owAtҭw�~�J�

```

***Exibe o cabeçalho do arquivo BAM ordenado utilizando o samtools view -H. O cabeçalho contém informações essenciais, como genoma de referência, contigs, grupos de leitura (Read Groups) e versões das ferramentas, sendo importante para validar a consistência dos metadados antes das análises subsequentes.***

```bash
MeuDrive="/content/drive/MyDrive/TRABALHO_FINAL"
SAMPLE="cap-ngse-b-2019"

samtools view -H "$MeuDrive/dados/bam/$SAMPLE.sorted.bam"
```

***Output:***

```
@HD	VN:1.6	SO:coordinate
@SQ	SN:chr10	LN:135534747
@RG	ID:cap-ngse-b-2019	SM:cap-ngse-b-2019	LB:Exoma	PL:Illumina
@PG	ID:bwa	PN:bwa	VN:0.7.17-r1188	CL:bwa mem -K 100000000 -R @RG\tID:cap-ngse-b-2019\tSM:cap-ngse-b-2019\tLB:Exoma\tPL:Illumina /content/drive/MyDrive/TRABALHO_FINAL/referencia/hg19/hg19.fasta /content/drive/MyDrive/TRABALHO_FINAL/dados/fastq/cap-ngse-b-2019-chr10_S1_L001_R1_001.fastq /content/drive/MyDrive/TRABALHO_FINAL/dados/fastq/cap-ngse-b-2019-chr10_S1_L001_R2_001.fastq
@PG	ID:samtools	PN:samtools	PP:bwa	VN:1.13	CL:samtools sort -O bam -o /content/drive/MyDrive/TRABALHO_FINAL/dados/bam/cap-ngse-b-2019.sorted.bam /content/drive/MyDrive/TRABALHO_FINAL/dados/bam/cap-ngse-b-2019.sam
@PG	ID:samtools.1	PN:samtools	PP:samtools	VN:1.13	CL:samtools view -H /content/drive/MyDrive/TRABALHO_FINAL/dados/bam/cap-ngse-b-2019.sorted.bam
```

***Este script exibe os primeiros alinhamentos do arquivo BAM ordenado, após converter o conteúdo binário para o formato SAM com samtools view. Além disso, apresenta um resumo das colunas do formato SAM/BAM, facilitando a interpretação dos campos de alinhamento. Essa etapa é utilizada para verificar a qualidade e a coerência dos alinhamentos antes de prosseguir.***


```bash
MeuDrive="/content/drive/MyDrive/TRABALHO_FINAL"
SAMPLE="cap-ngse-b-2019"

echo "🔍 Conteúdo do arquivo BAM (primeiros 10 alinhamentos):"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

echo "📊 Colunas do formato SAM/BAM:"
echo "1. QNAME: Nome do read"
echo "2. FLAG: Informações binárias (paired, mapped, etc)"
echo "3. RNAME: Cromossomo de referência"
echo "4. POS: Posição no cromossomo (1-based)"
echo "5. MAPQ: Qualidade do mapeamento"
echo "6. CIGAR: Descrição do alinhamento"
echo "7. RNEXT: Cromossomo do par (paired-end)"
echo "8. PNEXT: Posição do par"
echo "9. TLEN: Tamanho do fragmento"
echo "10. SEQ: Sequência do read"
echo "11. QUAL: Qualidades da sequência"
echo ""

echo "📄 Primeiros 10 alinhamentos:"
samtools view "$MeuDrive/dados/bam/$SAMPLE.sorted.bam" | head -10

```

***Converte o arquivo BAM ordenado para o formato BED utilizando o BEDTools, representando os alinhamentos como intervalos genômicos. Em seguida, realiza a fusão (merge) de regiões sobrepostas e a ordenação dos intervalos resultantes.
Essa etapa é utilizada para resumir e organizar as regiões genômicas cobertas pelas leituras, facilitando análises baseadas em intervalos, como avaliação de cobertura e interseção com regiões alvo.***

```bash
MeuDrive="/content/drive/MyDrive/TRABALHO_FINAL"
SAMPLE="cap-ngse-b-2019"

bedtools bamtobed -i "$MeuDrive/dados/bam/$SAMPLE.sorted.bam" > "$MeuDrive/dados/bam/$SAMPLE.bed"
bedtools merge -i "$MeuDrive/dados/bam/$SAMPLE.bed" > "$MeuDrive/dados/bam/$SAMPLE.merged.bed"
bedtools sort -i "$MeuDrive/dados/bam/$SAMPLE.merged.bed" > "$MeuDrive/dados/bam/$SAMPLE.sorted.bed"
```

***Exibe as dez primeiras linhas do arquivo BED ordenado, permitindo verificar os intervalos genômicos derivados dos alinhamentos (cromossomo, início e fim). Essa inspeção confirma que a conversão do BAM para BED e a organização das regiões foram realizadas corretamente.***


