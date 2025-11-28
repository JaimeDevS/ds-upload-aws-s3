
# Upload de arquivos na AWS S3

<img width="317" height="159" alt="image" src="https://hermes.dio.me/articles/cover/77837398-793e-413b-bc2b-bad53e9032e0.png" />


## Criar um Bucket 
bds-aula
us-east-1
Bloquear todo acesso público - desmarcar

## Entrar no Bucket

- Na aba permissões -> Política do bucket -> Editar -> Salvar
> OBS. colocar o nome do Bucket "Resource": " arn:aws:s3:::bds-aula/* "
```js
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::bds-aula/*"
        }
    ]
}
```

## CORS configuration para bucket
Na aba permissões -> Compartilhamento de recursos de origem cruzada (CORS) -> Salvar
```js
[
    {
        "AllowedHeaders": [
            "*"
        ],
        "AllowedMethods": [
            "GET",
            "HEAD"
        ],
        "AllowedOrigins": [
            "*"
        ],
        "ExposeHeaders": [],
        "MaxAgeSeconds": 3000
    }
]
```
## Códigos padrão
OBS. está sendo usada variáveis de ambiente para externalizar os dados sensíveis
```
aws.access_key_id=${AWS_KEY:empty}
aws.secret_access_key=${AWS_SECRET:empty}
s3.bucket=${DSCATALOG_BUCKET_NAME:empty}
s3.region=${DSCATALOG_BUCKET_REGION:sa-east-1}

spring.servlet.multipart.max-file-size=${MAX_FILE_SIZE:10MB}
spring.servlet.multipart.max-request-size=${MAX_FILE_SIZE:10MB}
```

## Grupo de permissões AWS
- criar grupo de permissões AULA com as seguintes permissões <br/>
AmazonS3FullAccess
AmazonEC2FullAccess
- criar um usuário (acesso programático) vinculando o grupo AULA
OBS. baixar as credencias gerada pela AWS no momento da criação do usuário <br/>

pegar:
- nome do bucket
- região
- as credenciais
	- access_key_id
	- access_key
	
#### Definir as variáveis de ambiente no S.O.

```java
@Service
public class S3Service {

	private static Logger LOG = LoggerFactory.getLogger(S3Service.class);

	@Autowired
	private AmazonS3 s3client;

	@Value("${s3.bucket}")
	private String bucketName;

	public void uploadFile(String localFilePath) {
		try {
			File file = new File(localFilePath);
			LOG.info("Upload start");
			s3client.putObject(new PutObjectRequest(bucketName, "test.jpg", file));
			LOG.info("Upload end");
		}
		catch (AmazonServiceException e) {
			LOG.info("AmazonServiceException: " + e.getErrorMessage());
			LOG.info("Status code: " + e.getErrorCode());
		}
		catch (AmazonClientException e) {
			LOG.info("AmazonClientException: " +  e.getMessage());
		}
	}
}
```
```java
@Configuration
public class S3Config {

	@Value("${aws.access_key_id}")
	private String awsId;

	@Value("${aws.secret_access_key}")
	private String awsKey;

	@Value("${s3.region}")
	private String region;

	@Bean
	public AmazonS3 s3client() {
		BasicAWSCredentials awsCred = new BasicAWSCredentials(awsId, awsKey);
		AmazonS3 s3client = AmazonS3ClientBuilder.standard().withRegion(Regions.fromName(region))
							.withCredentials(new AWSStaticCredentialsProvider(awsCred)).build();
		return s3client;
	}
}
```
