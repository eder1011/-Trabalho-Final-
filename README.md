
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










