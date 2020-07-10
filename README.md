# O que é SOLID?

## :pushin: Tabela de conteúdo

- SRP - Single Responsability Principle

## [SRP - Single Responsability Principle - Princípio da Responsabilidade Única](https://en.wikipedia.org/wiki/Single-responsibility_principle)

Diz que uma classe deve ter um único motivo para mudar, e uma única responsabilidade dentro do sistema. As God Class (class Deus), são um exemplo de um tipo de classe que realizam diversas tarefas, tendo assim mais de um motivo para manutenção e são comumente construídas quando estamos aprendendo programação orientada a objetos. Vejamos um exemplo de uma classe que possui duas responsabilidades, violando assim o princípio SRP baseado em um sistema de filas de e-mail.

```js
interface Job {
  jobName: string
  status: 'start' | 'progress' | 'failed' | 'done'
  jobAction?: Function
}

interface DTOMail {
  from: string
  to: string
  body: string
}

export class QueueSendMail {
  private queue: Job[] = []

  public async process (jobName: string, mail: DTOMail[]) {
    this.queue.push({ jobName, status: 'start' })
    mail.forEach(process => this.sendMail(process))
  }

  private async sendMail ({ from, to, body }: DTOMail) {
    this.queue.forEach(mail => {
      // factory to create new email
      mail.jobAction = () => {
        mail.status = 'progress'
        console.log(`Sending mail to ${to} by ${from} with content: ${body}`)
      }

      mail.jobAction()
      mail.status = 'done'
    })
  }
}

const queueMails = new QueueSendMail()

queueMails.process('SendMailToSubscribers', [
  {
    to: 'tha.motog@gmail.com',
    from: 'thalysonrodrigues.dev@gmail.com',
    body: 'Hello World!'
  },
  {
    to: 'tha.motog@gmail.com',
    from: 'thalyson@gmail.com',
    body: 'Hello World 2!'
  }
])
```

Podemos observar que a classe `QueueSendMail` tem duas responsabilidades: criar uma fila para envio de e-mails e enviar estes e-mails. Além disto, existem dois motivos pelos quais a classe pode ser alterada, a forma como armazena e configura a fila e como envia seus e-mails. Podemos melhorar esta classe aplicando o conceito de SRP (Single Responsability Principle). Vejamos:

```js
export class Queue {
  private queue: Job[] = []

  public async process (jobName: string, jobAction: (data: any, done?: () => void) => void) {
    this.queue.push({ jobName, jobAction, status: 'start' })
  }

  public async create (jobName: string, data: any) {
    const job = this.queue.find(job => job.jobName === jobName)

    if (job && job.jobAction) {
      job.jobAction(data)
      job.status = 'done'

      return true
    }

    return false
  }
}

export class SendMail {
  public async sendMail ({ from, to, body }: DTOMail) {
    console.log(`Sending mail to ${to} by ${from} with content: ${body}`)
  }
}

const queue = new Queue()
const mailer = new SendMail()

queue.process('SendMailToSubscribers', async (data, done) => {
  await mailer.sendMail(data)
  return done && done()
})

queue.create('SendMailToSubscribers', {
  to: 'tha.devg@gmail.com',
  from: 'thalyson@gmail.com',
  body: 'Hello World 3!'
})
```

Agora as responsabilidades foram divididas. A classe Queue é reponsável por registrar (método `process`) e criar (método `create`) jobs para serem processados em background. O que faz ser possível agora criar outros tipos de processamentos assíncronos, não apenas enviar e-mails. A classe `SendMail` agora é responsável apenas por enviar e-mails, e a comunicação entre estes requisitos é feita entre objetos e não em classe (god class). Podemos perceber a presença do [Pattern Callback](https://en.wikipedia.org/wiki/Callback_(computer_programming)) para realizar tal comunicação entre os objetos criados. Claramente estas são soluções fakes criadas apenas para ilustrar um problema do mundo real e foram inspiradas em na [lib Kue](https://github.com/Automattic/kue).
