# SD2_Cli_Srv_API_Instagram

API REST para utilização na matéria AD36A - Tecnologias Cliente Servidor - AS61 (ou Sistemas Distribuidos 2).

Código aberto para possibilidade de ajustes e colaboração de todos.

## Visao Geral

Esta documentacao resume os requisitos funcionais e nao funcionais definidos em aula organizando os recursos principais da API:

- Usuarios
- Autenticacao
- Postagens
- Curtidas
- Comentarios
- Seguidores
- Feed
- Perfil

## Pontos a definir ainda

- Não foi debatida a utilização de códigos identificadores para os objetos, levantar a necessidade 

- Anteriormente decidido permitir ou não permitir caracteres especiais pra diferentes operações, avaliar se não vai apenas complicar algumas funções
- Imagens limitadas a 10Mb, o sistema irá recusar ou ajustar o tamanho para obedecer o limite?

- As imagens serao referencias online, ou serão salvas no servidor? Se no servidor, armazenar imagem em banco ou apenas referencia ao caminho da imagem?  Se no cliente, enviar apenas referencia relativa?

- Os IDs estarão explicitos na URL, ou utilizaremos UUID, ULID, slug, ou outra solução?
    - UUID: GET /usuarios/550e8400-e29b-41d4-a716-446655440000
    - ULID: GET /usuarios/01HZX3J8Z7A9Q5K8R2N6T4M1XW
    - slug: GET /usuarios/joaosilva


## Padrao da API

- Arquitetura: cliente-servidor
- Protocolo: HTTP
- Estilo: REST
- Formato padrao de requisicao e resposta: JSON
- Autenticacao: Bearer Token (JWT??)
- Upload de imagem: `multipart/form-data`

### Codigos HTTP esperados

- `200 OK`
- `201 Created`
- `400 Bad Request`
- `401 Unauthorized`
- `404 Not Found`

## Requisitos Funcionais



### Usuarios

- Cadastro de usuario com nome completo, username, e-mail e senha
- Validacao de nome entre 3 e 60 caracteres
- Validacao de username entre 3 e 30 caracteres e unico no sistema
- Validacao de e-mail com formato valido e unico
- Validacao de senha entre 8 e 24 caracteres

### Autenticacao

- Login com username e senha
- Logout de sessao

### Postagens

- Criacao de post com 1 imagem obrigatoria
- Legenda opcional com ate 200 caracteres
- Upload aceitando JPG, JPEG ou PNG
- Tamanho maximo de 10 MB
- Edicao apenas da legenda
- Exclusao de postagem propria

### Curtidas

- Curtir postagem autenticado
- Remover curtida
- Exibir contagem de curtidas

### Comentarios

- Criar comentario com 1 a 300 caracteres
- Excluir comentario proprio
- Permitir moderacao pelo dono do post

### Seguidores

- Seguir outros usuarios
- Deixar de seguir
- Impedir que o usuario siga a si mesmo
- Exibir contadores de seguidores e seguindo
- Listar seguidores e seguindo

### Feed

- Exibir postagens dos usuarios seguidos

### Perfil

- Editar nome, username, foto de perfil e bio
- Visualizar perfis publicos

## Requisitos Nao Funcionais

- API REST via HTTP
- Comunicacao em JSON
- Rotas protegidas com autenticacao
- Uso adequado dos metodos HTTP
- Upload de arquivos via `multipart/form-data`

## Endpoints

### Usuarios

| Metodo | Rota | Descricao |
| --- | --- | --- |
| POST | `/usuarios` | Cadastrar usuario |
| GET | `/usuarios/{id}` | Buscar usuario por ID |
| PUT | `/usuarios/{id}` | Atualizar usuario |

### Autenticacao

| Metodo | Rota | Descricao |
| --- | --- | --- |
| POST | `/auth/login` | Realizar login |
| POST | `/auth/logout` | Realizar logout |

### Postagens

| Metodo | Rota | Descricao |
| --- | --- | --- |
| POST | `/postagens` | Criar postagem |
| GET | `/postagens/{id}` | Buscar postagem |
| PUT | `/postagens/{id}` | Editar legenda |
| DELETE | `/postagens/{id}` | Excluir postagem |

### Curtidas

| Metodo | Rota | Descricao |
| --- | --- | --- |
| POST | `/postagens/{id}/curtir` | Curtir postagem |
| DELETE | `/postagens/{id}/curtir` | Remover curtida |

### Comentarios

| Metodo | Rota | Descricao |
| --- | --- | --- |
| POST | `/postagens/{id}/comentarios` | Criar comentario |
| DELETE | `/comentarios/{id}` | Excluir comentario |

### Seguidores

| Metodo | Rota | Descricao |
| --- | --- | --- |
| POST | `/usuarios/{id}/seguir` | Seguir usuario |
| DELETE | `/usuarios/{id}/seguir` | Deixar de seguir |
| GET | `/usuarios/{id}/seguidores` | Listar seguidores |
| GET | `/usuarios/{id}/seguindo` | Listar quem o usuario segue |

### Feed

| Metodo | Rota | Descricao |
| --- | --- | --- |
| GET | `/feed` | Listar postagens do feed |

### Perfil

| Metodo | Rota | Descricao |
| --- | --- | --- |
| GET | `/perfil/{id}` | Visualizar perfil |
| PUT | `/perfil/{id}` | Editar perfil |

## Autenticacao

As rotas protegidas devem receber o token no cabecalho:

```http
Authorization: Bearer seu_token_aqui
```

## Contrato Padrao de Resposta

A API deve usar o mesmo envelope JSON em sucesso e erro.

### Exemplo resposta de sucesso

```json
{
	"success": true,
	"message": "Operacao realizada com sucesso",
	"data": {},
	"meta": null
}
```

- `success`: indica se a operacao foi concluida com sucesso
- `message`: mensagem resumida para o cliente
- `data`: payload principal da resposta
- `meta`: informacoes auxiliares, como paginacao, quando aplicavel

### Exemplo resposta de erro

```json
{
	"success": false,
	"message": "Falha ao processar a requisicao",
	"error": {
		"code": "VALIDATION_ERROR",
		"details": [
			{
				"field": "email",
				"message": "E-mail ja cadastrado"
			}
		]
	},
	"meta": null
}
```

- `error.code`: codigo interno do erro
- `error.details`: lista de campos e mensagens quando houver validacao

### Codigos de erro sugeridos

- `VALIDATION_ERROR`
- `UNAUTHORIZED`
- `FORBIDDEN`
- `NOT_FOUND`
- `CONFLICT`
- `UPLOAD_ERROR`

## Exemplos de Requisicao e Resposta JSON

Os exemplos abaixo seguem o contrato padrao. Para endpoints com upload de imagem, a requisicao real deve ser enviada em `multipart/form-data`, mas a resposta continua em JSON.

### 1. Cadastro de usuario

**POST** `/usuarios`

Requisicao:

```json
{
	"nome": "Professor Richard",
	"username": "professorrichard",
	"email": "professor@email.com",
	"senha": "SenhaForte123"
}
```

Resposta `201 Created`:

```json
{
	"success": true,
	"message": "Usuario cadastrado com sucesso",
	"data": {
		"id": 1,
		"nome": "Professor Richard",
		"username": "professorrichard",
		"email": "professor@email.com",
		"criadoEm": "2026-04-09T10:00:00Z"
	},
	"meta": null
}
```

Resposta `400 Bad Request`:

```json
{
	"success": false,
	"message": "Dados de cadastro invalidos",
	"error": {
		"code": "VALIDATION_ERROR",
		"details": [
			{
				"field": "email",
				"message": "E-mail ja cadastrado"
			},
			{
				"field": "senha",
				"message": "A senha deve ter entre 8 e 24 caracteres"
			}
		]
	},
	"meta": null
}
```

### 2. Buscar usuario por ID

**GET** `/usuarios/{id}`

Resposta `200 OK`:

```json
{
	"success": true,
	"message": "Usuario encontrado com sucesso",
	"data": {
		"id": 1,
		"nome": "Professor Richard",
		"username": "professorrichard",
		"email": "professor@email.com",
		"bio": "Criador de conteudo",
		"fotoPerfil": "https://api.exemplo.com/uploads/perfis/1.jpg",
		"seguidores": 120,
		"seguindo": 89
	},
	"meta": null
}
```

Resposta `404 Not Found`:

```json
{
	"success": false,
	"message": "Usuario nao encontrado",
	"error": {
		"code": "NOT_FOUND",
		"details": [
			{
				"field": "id",
				"message": "Nenhum usuario foi localizado com o id informado"
			}
		]
	},
	"meta": null
}
```

### 3. Atualizar usuario

**PUT** `/usuarios/{id}`

Requisicao:

```json
{
	"nome": "Novo Professor",
	"username": "novoprofessor",
	"email": "novoprofessor@email.com"
}
```

Resposta `200 OK`:

```json
{
	"success": true,
	"message": "Usuario atualizado com sucesso",
	"data": {
		"id": 1,
		"nome": "Novo Professor",
		"username": "novoprofessor",
		"email": "novoprofessor@email.com"
	},
	"meta": null
}
```

### 4. Login

**POST** `/auth/login`

Requisicao:

```json
{
	"username": "professorrichard",
	"senha": "SenhaForte123"
}
```

Resposta `200 OK`:

```json
{
	"success": true,
	"message": "Login realizado com sucesso",
	"data": {
		"token": "eyJhbGciOi...",
		"tipo": "Bearer",
		"usuario": {
			"id": 1,
			"nome": "Professor Richard",
			"username": "professorrichard"
		}
	},
	"meta": null
}
```

Resposta `401 Unauthorized`:

```json
{
	"success": false,
	"message": "Credenciais invalidas",
	"error": {
		"code": "UNAUTHORIZED",
		"details": [
			{
				"field": "username",
				"message": "Username ou senha invalidos"
			}
		]
	},
	"meta": null
}
```

### 5. Logout

**POST** `/auth/logout`

Resposta `200 OK`:

```json
{
	"success": true,
	"message": "Logout realizado com sucesso",
	"data": null,
	"meta": null
}
```

Resposta `401 Unauthorized`:

```json
{
	"success": false,
	"message": "Token invalido ou expirado",
	"error": {
		"code": "UNAUTHORIZED",
		"details": [
			{
				"field": "Authorization",
				"message": "Informe um Bearer Token valido"
			}
		]
	},
	"meta": null
}
```

### 6. Criar postagem

**POST** `/postagens`

Observacao: a requisicao deve ser enviada em `multipart/form-data` com a imagem e, opcionalmente, a legenda.

Campos conceituais enviados:

```json
{
	"legenda": "Primeira foto no app"
}
```

Resposta `201 Created`:

```json
{
	"success": true,
	"message": "Postagem criada com sucesso",
	"data": {
		"id": 15,
		"autorId": 1,
		"imagemUrl": "https://api.exemplo.com/uploads/postagens/15.jpg",
		"legenda": "Primeira foto no app",
		"curtidas": 0,
		"comentarios": 0,
		"criadoEm": "2026-04-09T10:30:00Z"
	},
	"meta": null
}
```

Resposta `400 Bad Request`:

```json
{
	"success": false,
	"message": "Falha no upload da postagem",
	"error": {
		"code": "UPLOAD_ERROR",
		"details": [
			{
				"field": "imagem",
				"message": "A imagem deve estar nos formatos JPG, JPEG ou PNG e ter no maximo 10 MB"
			}
		]
	},
	"meta": null
}
```

### 7. Buscar postagem

**GET** `/postagens/{id}`

Resposta `200 OK`:

```json
{
	"success": true,
	"message": "Postagem encontrada com sucesso",
	"data": {
		"id": 15,
		"autor": {
			"id": 1,
			"username": "professorrichard",
			"fotoPerfil": "https://api.exemplo.com/uploads/perfis/1.jpg"
		},
		"imagemUrl": "https://api.exemplo.com/uploads/postagens/15.jpg",
		"legenda": "Primeira foto no app",
		"curtidas": 3,
		"comentarios": [
			{
				"id": 101,
				"autorUsername": "maria",
				"texto": "Muito boa essa foto"
			}
		],
		"criadoEm": "2026-04-09T10:30:00Z"
	},
	"meta": null
}
```

### 8. Editar legenda da postagem

**PUT** `/postagens/{id}`

Requisicao:

```json
{
	"legenda": "Legenda atualizada da postagem"
}
```

Resposta `200 OK`:

```json
{
	"success": true,
	"message": "Postagem atualizada com sucesso",
	"data": {
		"id": 15,
		"legenda": "Legenda atualizada da postagem"
	},
	"meta": null
}
```

### 9. Excluir postagem

**DELETE** `/postagens/{id}`

Resposta `200 OK`:

```json
{
	"success": true,
	"message": "Postagem excluida com sucesso",
	"data": null,
	"meta": null
}
```

Resposta `404 Not Found`:

```json
{
	"success": false,
	"message": "Postagem nao encontrada",
	"error": {
		"code": "NOT_FOUND",
		"details": [
			{
				"field": "id",
				"message": "Nenhuma postagem foi localizada para o id informado"
			}
		]
	},
	"meta": null
}
```

### 10. Curtir postagem

**POST** `/postagens/{id}/curtir`

Resposta `200 OK`:

```json
{
	"success": true,
	"message": "Postagem curtida com sucesso",
	"data": {
		"postagemId": 15,
		"curtidas": 4
	},
	"meta": null
}
```

### 11. Remover curtida

**DELETE** `/postagens/{id}/curtir`

Resposta `200 OK`:

```json
{
	"success": true,
	"message": "Curtida removida com sucesso",
	"data": {
		"postagemId": 15,
		"curtidas": 3
	},
	"meta": null
}
```

### 12. Criar comentario

**POST** `/postagens/{id}/comentarios`

Requisicao:

```json
{
	"texto": "Muito boa essa foto"
}
```

Resposta `201 Created`:

```json
{
	"success": true,
	"message": "Comentario criado com sucesso",
	"data": {
		"id": 101,
		"postagemId": 15,
		"autorId": 2,
		"texto": "Muito boa essa foto",
		"criadoEm": "2026-04-09T10:45:00Z"
	},
	"meta": null
}
```

### 13. Excluir comentario

**DELETE** `/comentarios/{id}`

Resposta `200 OK`:

```json
{
	"success": true,
	"message": "Comentario excluido com sucesso",
	"data": null,
	"meta": null
}
```

Resposta `404 Not Found`:

```json
{
	"success": false,
	"message": "Comentario nao encontrado",
	"error": {
		"code": "NOT_FOUND",
		"details": [
			{
				"field": "id",
				"message": "Nenhum comentario foi localizado para o id informado"
			}
		]
	},
	"meta": null
}
```

### 14. Seguir usuario

**POST** `/usuarios/{id}/seguir`

Resposta `200 OK`:

```json
{
	"success": true,
	"message": "Agora voce esta seguindo este usuario",
	"data": {
		"usuarioId": 2,
		"seguindo": true
	},
	"meta": null
}
```

Resposta `400 Bad Request`:

```json
{
	"success": false,
	"message": "Operacao de seguir invalida",
	"error": {
		"code": "VALIDATION_ERROR",
		"details": [
			{
				"field": "id",
				"message": "Nao e permitido seguir a si mesmo"
			}
		]
	},
	"meta": null
}
```

### 15. Deixar de seguir

**DELETE** `/usuarios/{id}/seguir`

Resposta `200 OK`:

```json
{
	"success": true,
	"message": "Voce deixou de seguir este usuario",
	"data": {
		"usuarioId": 2,
		"seguindo": false
	},
	"meta": null
}
```

### 16. Listar seguidores

**GET** `/usuarios/{id}/seguidores`

Resposta `200 OK`:

```json
{
	"success": true,
	"message": "Seguidores listados com sucesso",
	"data": [
		{
			"id": 2,
			"username": "maria",
			"fotoPerfil": "https://api.exemplo.com/uploads/perfis/2.jpg"
		},
		{
			"id": 3,
			"username": "ana",
			"fotoPerfil": "https://api.exemplo.com/uploads/perfis/3.jpg"
		}
	],
	"meta": {
		"pagina": 1,
		"limite": 10,
		"total": 2
	}
}
```

### 17. Listar seguindo

**GET** `/usuarios/{id}/seguindo`

Resposta `200 OK`:

```json
{
	"success": true,
	"message": "Usuarios seguidos listados com sucesso",
	"data": [
		{
			"id": 4,
			"username": "carlos",
			"fotoPerfil": "https://api.exemplo.com/uploads/perfis/4.jpg"
		},
		{
			"id": 5,
			"username": "beatriz",
			"fotoPerfil": "https://api.exemplo.com/uploads/perfis/5.jpg"
		}
	],
	"meta": {
		"pagina": 1,
		"limite": 10,
		"total": 2
	}
}
```

### 18. Listar feed

**GET** `/feed`

Resposta `200 OK`:

```json
{
	"success": true,
	"message": "Feed carregado com sucesso",
	"data": [
		{
			"id": 15,
			"autor": {
				"id": 1,
				"username": "professorrichard"
			},
			"imagemUrl": "https://api.exemplo.com/uploads/postagens/15.jpg",
			"legenda": "Legenda atualizada da postagem",
			"curtidas": 4,
			"comentarios": 1,
			"criadoEm": "2026-04-09T10:30:00Z"
		},
		{
			"id": 16,
			"autor": {
				"id": 3,
				"username": "ana"
			},
			"imagemUrl": "https://api.exemplo.com/uploads/postagens/16.png",
			"legenda": "Bom dia",
			"curtidas": 10,
			"comentarios": 2,
			"criadoEm": "2026-04-09T09:00:00Z"
		}
	],
	"meta": {
		"pagina": 1,
		"limite": 10,
		"total": 2
	}
}
```

### 19. Visualizar perfil

**GET** `/perfil/{id}`

Resposta `200 OK`:

```json
{
	"success": true,
	"message": "Perfil carregado com sucesso",
	"data": {
		"id": 1,
		"nome": "Novo Professor",
		"username": "novoprofessor",
		"bio": "Criador de conteudo",
		"fotoPerfil": "https://api.exemplo.com/uploads/perfis/1.jpg",
		"seguidores": 120,
		"seguindo": 89,
		"totalPostagens": 34
	},
	"meta": null
}
```

### 20. Editar perfil

**PUT** `/perfil/{id}`

Requisicao:

```json
{
	"nome": "Novo Professor",
	"username": "novoprofessor",
	"bio": "Criador de conteudo e apaixonado por fotografia",
	"fotoPerfil": "https://api.exemplo.com/uploads/perfis/1-nova.jpg"
}
```

Resposta `200 OK`:

```json
{
	"success": true,
	"message": "Perfil atualizado com sucesso",
	"data": {
		"id": 1,
		"nome": "Novo Professor",
		"username": "novoprofessor",
		"bio": "Criador de conteudo e apaixonado por fotografia",
		"fotoPerfil": "https://api.exemplo.com/uploads/perfis/1-nova.jpg"
	},
	"meta": null
}
```

## Observacoes para Implementacao

- Use sempre o mesmo envelope com `success`, `message`, `data` e `meta`
- Em erros, retorne sempre `success: false` com `error.code` e `error.details`
- Considere paginacao em endpoints de listagem, como feed, seguidores e comentarios
- Em endpoints protegidos, retorne `401 Unauthorized` quando o token estiver ausente ou invalido
- Em recursos inexistentes, retorne `404 Not Found`
