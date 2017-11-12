## NodeJS Files

## index.js
```javascript
const express = require('express')
const app = express()
const port = process.env.PORT || 3000
const router = require('./routes')

router(app)

app.listen(port, () => {
  console.log(`Server running at port:${port}`)
})
```

## routes index.js
```javascript
const PostsRouter = require('./PostsRouter')

module.exports = app => {
  app.use('/post', PostsRouter)
  app.use('*', (req, res) => res.status(404).send({msg: '404!'}))
}
```

## routes PostsRouter.js
```javascript
const Router = require('express').Router()

const PostsController = require('../controllers/PostsController')

Router.get('/', PostsController.get)
Router.get('/create', PostsController.create)
Router.get('/:id/edit', PostsController.update)
Router.get('/:id/delete', PostsController.destroy)
Router.get('/:id', PostsController.getById)

module.exports = Router
```

## constrollers PostsController.js
```javascript
const fs = require('fs')
const path = require('path')
const uuidv1 = require('uuid/v1');
const faker = require('faker');
const moment = require('moment')
const time = moment()
const posts = path.resolve(__dirname, '../databases/files/posts.txt')

const get = (req, res) => {
  fs.readFile(posts, 'utf-8', (err, data) => {
    if (err) {
      console.error(err)
      return
    }

    res.status(200).json(JSON.parse(data))
  })
}

const create = (req, res) => {
  const newPost = {
    id: uuidv1(),
    topic: faker.lorem.sentence(),
    content: faker.lorem.paragraph(),
    created_at: time.format(),
    updated_at: time.format(),
    owner_id: faker.random.number(100)
  }

  fs.readFile(posts, 'utf-8', (err, data) => {
    if (err) {
      console.error(err)
      return
    }

    const currentData = JSON.parse(data)

    fs.writeFile(posts, JSON.stringify([...currentData, newPost]), (err) => {
      if (err) {
        console.error(err)
        return
      }

      res.status(200).json({message: 'success'})
    })
  })
}

const update = (req, res) => {
  fs.readFile(posts, 'utf-8', (err, data) => {
    if (err) {
      console.error(err)
      return
    }

    const currentData = JSON.parse(data)
    const currentPost = currentData.find(e => e.id === req.params.id)

    currentPost.topic = 'edit topic'
    currentPost.content = 'edit content'
    currentPost.updated_at = time.format()

    fs.writeFile(posts, JSON.stringify([...currentData.filter(e => e.id !== req.params.id), currentPost]), (err) => {
      if (err) {
        console.error(err)
        return
      }

      res.status(200).json(currentPost)
    })
  })
}

const destroy = (req, res) => {
  fs.readFile(posts, 'utf-8', (err, data) => {
    if (err) {
      console.error(err)
      return
    }

    const currentData = JSON.parse(data)
    const currentPost = currentData.find(e => e.id === req.params.id)

    fs.writeFile(posts, JSON.stringify([...currentData.filter(e => e.id !== req.params.id)]), (err) => {
      if (err) {
        console.error(err)
        return
      }

      res.status(200).json({message: 'success'})
    })
  })
}

const getById = (req, res) => {
  fs.readFile(posts, 'utf-8', (err, data) => {
    if (err) {
      console.error(err)
      return
    }

    const currentData = JSON.parse(data)
    const result = currentData.find(e => e.id === req.params.id)

    if (!result) res.status(404).json({status: 404, message: 'page not found'})

    res.status(200).json(result)
  })
}

module.exports = {
  get,
  create,
  update,
  destroy,
  getById
}
```