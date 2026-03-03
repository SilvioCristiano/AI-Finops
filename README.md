# AI-Finops
Este é o arquivo `README.md` atualizado e formatado para o seu projeto, refletindo as mudanças de arquitetura para autenticação via OAuth e integração com a API ORDS.

---

# OCI Function: CSV to ORDS API Loader

Esta função Oracle Cloud (Fn Project) automatiza o processamento de arquivos CSV depositados no **Object Storage**, convertendo cada linha em um objeto JSON e enviando-o para uma API REST (ORDS) protegida por **OAuth2**.

## 💡 Arquitetura do Sistema

Abaixo, a representação visual do fluxo de dados:

```ascii
    +-------------+          +----------------+          +-----------------+
    |   Usuário   |          | Object Storage |          |   Cloud Events  |
    |  (Upload)   |--------->| (input-bucket) |--------->| (Object Create) |
    +-------------+          +----------------+          +--------+--------+
                                                                  |
                                                                  v
    +-------------------------------------------------------------+---------+
    |                         OCI FUNCTION                                  |
    |                                                                       |
    |  1. Obtém Bearer Token (OAuth2)                                       |
    |  2. Lê arquivo CSV do bucket                                          |
    |  3. Para cada linha: POST JSON -> ORDS API                            |
    |  4. Move arquivo: input-bucket -> processed-bucket                    |
    +------------------------------+----------------------------------------+
                                   |
              +--------------------+---------------------+
              |                                          |
              v                                          v
    +-------------------+                    +---------------------------+
    |  Identity (IAM)   |                    |   Autonomous Database     |
    | (Auth & Secrets)  |                    |      (ORDS / APEX)        |
    +-------------------+                    +---------------------------+

```

---

## 🚀 Como Funciona

1. **Gatilho:** Um arquivo CSV é carregado no `input-bucket`.
2. **Evento:** O OCI Events identifica o upload e dispara esta Function.
3. **Processamento:**
* A função solicita um token de acesso via `client_credentials` à URL do OAuth.
* O CSV é lido via streaming.
* Cada linha é convertida em um JSON e enviada no corpo da requisição (`p_body`) para a API de destino.


4. **Finalização:** O arquivo original é movido para o `processed-bucket` para evitar reprocessamento.

---

## 🛠️ Pré-requisitos

* **OCI CLI** configurado e permissões de **Resource Principal**.
* **Dynamic Group** incluindo o OCID da função ou do compartimento.
* **Políticas de IAM** para permitir que a função gerencie objetos nos buckets.
* **API ORDS** configurada com OAuth2 Client Credentials.

---

## ⚙️ Configuração (App Context)

A função espera as seguintes variáveis de configuração no OCI Functions:

| Chave | Descrição | Exemplo |
| --- | --- | --- |
| `input-bucket` | Nome do bucket de entrada | `csv-uploads` |
| `processed-bucket` | Nome do bucket de destino (pós-processamento) | `csv-processed` |
| `ords-token-url` | URL do endpoint OAuth2 | `https://.../ords/admin/oauth/token` |
| `ords-api-url` | URL da API que receberá os dados | `https://.../ords/admin/insert_costanalisys/` |
| `ords-client-id` | Client ID do OAuth | `XYZ123...` |
| `ords-client-secret` | Client Secret do OAuth | `secret-abc...` |

> [!IMPORTANT]
> Para maior segurança em produção, recomenda-se buscar o `ords-client-secret` via **OCI Vault** em vez de variável de ambiente.

---

## 📦 Implantação

1. Certifique-se de estar no diretório da função.
2. Faça o deploy usando o Fn CLI:

```bash
fn -v deploy --app <nome-da-sua-app>
function oci-load-file-into-adw-python oci-load-file-into-adw-python ords-token-url "https://<>.sa-saopaulo-1.oraclecloudapps.com/ords/admin/oauth/token"
function oci-load-file-into-adw-python oci-load-file-into-adw-python ords-api-url "https://<>.sa-saopaulo-1.oraclecloudapps.com/ords/admin/insert_costanalisys/"
function oci-load-file-into-adw-python oci-load-file-into-adw-python ords-client-secret 'QbB9111D-4g..'
function oci-load-file-into-adw-python oci-load-file-into-adw-python ords-client-id '15sbCOZM111w..'
function oci-load-file-into-adw-python oci-load-file-into-adw-python input-bucket "input-bucket"
function oci-load-file-into-adw-python oci-load-file-into-adw-python processed-bucket "processed-bucket"

```

---

## 📋 Exemplo de Payload de Saída (API ORDS)

A função enviará um POST para a `api_url` com a seguinte estrutura:

```json
{
  "p_body": "{\"coluna1\": \"valor\", \"coluna2\": \"valor\"}"
}

```

---

## 🔍 Monitoramento

Os logs podem ser visualizados através do **OCI Logging Service** associado à aplicação da função. Procure por mensagens prefixadas com `INFO` ou `ERROR` para depurar o processamento dos arquivos.

---

**Deseja que eu ajude a criar o arquivo `func.yaml` correspondente a essas configurações?**
