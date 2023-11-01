# Microsserviço de Encoder de Vídeo

Uma solução eficiente para codificação e gerenciamento de vídeos, construída em Go. Este serviço foi projetado para facilitar o upload, armazenamento, fragmentação e conversão de formatos de vídeos, tornando-o uma solução versátil para necessidades de codificação de vídeo.

## Origem do Projeto

Esse microsserviço de Encoder de Vídeo foi desenvolvido como parte das aulas do curso oferecido pela [FullCycle](https://fullcycle.com.br) como uma parte prática do curso. O projeto foi criado como um exemplo prático e uma aplicação real do conhecimento adquirido durante o curso.

Durante o curso, tive a oportunidade de aprender e aplicar os princípios de codificação de vídeo, arquitetura de microsserviços, integração com serviços de terceiros e muito mais.

## Configuração do Ambiente

Para iniciar o serviço em modo de desenvolvimento, siga estas etapas:

1. **Clone o Repositório**: Faça um clone deste repositório no seu ambiente local.
2. **Duplique o Arquivo `.env.example`**: Crie uma cópia do arquivo `.env.example` e renomeie-o para `.env`.
3. **Execute o Docker Compose**: Utilize o comando `docker-compose up -d` para iniciar os contêineres necessários.
4. **Configuração do RabbitMQ**:
   - Acesse a administração do RabbitMQ.
   - Crie uma exchange do tipo `fanout` para ser usada como "Dead Letter Exchange" para mensagens não processadas.
   - Crie uma "Dead Letter Queue" e vincule-a à "Dead Letter Exchange", sem necessidade de routing_key.
   - No arquivo `.env`, informe o nome da "Dead Letter Exchange" no parâmetro `RABBITMQ_DLX`.
5. **Credenciais do Google Cloud Storage**: Crie uma conta de serviço no Google Cloud Platform (GCP) com permissões de gravação no Google Cloud Storage. Baixe o arquivo JSON de credenciais e salve-o na raiz do projeto com o nome `bucket-credential.json`.

## Executando o Encoder

Para executar o encoder, utilize o comando `make server` diretamente no container. Por exemplo:

```bash
docker exec encoder-new2_app_1 make server
```

## Padrão de Envio de Mensagens para o Encoder

Para que o sistema de encoder possa processar uma mensagem, ela deve estar no seguinte formato JSON:

```
{
  "resource_id": "my-resource-id-can-be-a-uuid-type",
  "file_path": "convite.mp4"
}
```

- resource_id: Representa o ID do vídeo que deseja converter (do tipo string).
- file_path: É o caminho completo do vídeo MP4 no bucket.

## Padrão de Retorno de Mensagens pelo Encoder

### Sucesso no Processamento

Quando o processamento é concluído com sucesso, o encoder envia o resultado para uma exchange configurada no arquivo .env. O padrão de retorno em JSON é:

```
{
    "id": "bbbdd123-ad05-4dc8-a74c-0000000000",
    "output_bucket_path": "codeeducationtest",
    "status": "COMPLETED",
    "video": {
        "encoded_video_folder": "b3f2d41e-2c0a-4830-bd65-0000000000",
        "resource_id": "aadc5ff9-0b0d-13ab-4a40-000000000",
        "file_path": "convite.mp4"
    },
    "Error": "",
    "created_at": "0000-00-2700:43:34.850479-04:00",
    "updated_at": "0000-00-27T00:43:38.081754-04:00"
}
```

- encoded_video_folder é a pasta que contém o vídeo convertido.

### Erro no Processamento

Quando ocorre um erro no processamento, o padrão de retorno em JSON é:

```
{
    "message": {
        "resource_id": "aadc5ff9-010d-a3ab-4a40-0000000000",
        "file_path": "convite.mp4"
    },
    "error": "Motivo do erro"
}
```

Além disso, o encoder envia a mensagem original que encontrou um problema durante o processamento para uma "Dead Letter Exchange" configurada. Basta definir a DLX desejada no arquivo .env no parâmetro RABBITMQ_DLX.

## Contribuição

Contribuições são bem-vindas! Se você encontrar um bug ou tiver uma melhoria, sinta-se à vontade para abrir um problema ou enviar um pull request.

## Licença

Este projeto está licenciado sob a Licença MIT. Consulte o arquivo LICENSE para obter detalhes.
