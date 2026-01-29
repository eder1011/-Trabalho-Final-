
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












