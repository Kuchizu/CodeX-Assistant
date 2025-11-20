## check-description.yml

Описание

Workflow проверяет, заполнено ли описание пулл реквеста. В зависимости от режима может просто оставить комментарий, закрыть PR или перевести его в draft.

Режимы
- `comment` (по умолчанию) — только комментарий, статус проверки зеленый.
- `strict` — комментарий и закрытие PR.
- `draft` — комментарий и автоматический перевод PR в draft

Как работает
- Запускается как reusable workflow через `workflow_call`.  
- Использует `actions/github-script@v7`
- Получает pull request из `context.payload.pull_request`.
- Проверяет поле `body`: пустое или нет после `trim()`.
- Если описание есть, просто завершает работу.
- Если описания нет:
  - создает комментарий через `github.rest.issues.createComment` с текстом о пустом description;
  - в режиме `strict` вызывает `github.rest.pulls.update(..., state: 'closed')` и закрывает PR;
  - в режиме `draft` выполняет конвертацию `convertPullRequestToDraft` с `pullRequestId = pr.node_id`, чтобы перевести PR в draft.


## check-linked-issue.yml

Описание

Workflow проверяет, есть ли в описании пулл реквеста ссылка на issue. Если связанной задачи нет, в зависимости от режима может ограничиться комментарием, закрыть PR или перевести его в draft.

Режимы
- `comment` (по умолчанию) — только комментарий.
- `strict` — комментарий и закрытие PR.
- `draft` — комментарий и перевод PR в draft.

Как работает
- Запускается как reusable workflow через `workflow_call`.  
- Использует `actions/github-script@v7`
- Получает свежие данные PR через `github.rest.pulls.get`.
- Ищет в `body` конструкции вида `closes #123`, `fixes #45`, `resolves #10` и т.п. с помощью регулярного выражения по ключевым словам (`close|closes|closed|fix|fixes|fixed|resolve|resolves|resolved`) и номеру issue.
- Если хотя бы одна такая ссылка найдена, workflow завершает работу без дополнительных действий.
- Если ссылок нет:
  - создает комментарий через `github.rest.issues.createComment` с информацией о том, что PR не привязан к issue;
  - в режиме `strict` закрывает PR через `github.rest.pulls.update(..., state: 'closed')`;
  - в режиме `draft` переводит PR в draft через GraphQL-мутацию `convertPullRequestToDraft` по `pr.node_id`.
