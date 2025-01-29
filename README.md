# Pacman Unity Clone

## 📌 Introduction
The **Pacman Unity Clone** is a recreation of the classic arcade game Pac-Man, built using Unity. This project implements character movement, ghost AI, pellet collection, power-ups, and a simple passage system for teleportation across the map.

![Pacman](https://user-images.githubusercontent.com/62818241/210207502-c8399d99-94a5-4f89-85b1-eb15fc7805ad.PNG)

## 🔥 Features
- 🎮 **Player Movement**: Move Pacman using arrow keys or WASD.
- 👻 **Ghost AI**: Ghosts roam and chase Pacman using different behaviors.
- 🍏 **Pellet Collection**: Pacman eats pellets for points.
- ⚡ **Power Pellets**: Temporary ability to eat ghosts.
- 🔄 **Passage System**: Allows teleporting from one side of the map to the other.
- 🏗 **Node-Based Navigation**: Ghosts move along predefined paths.

---

## 🏗️ How It Works

The game is structured with various scripts managing movement, AI behavior, collision detection, and interactions.

### 📌 **Ghost Scatter Behavior**
Controls how ghosts move when in scatter mode.

```csharp
using UnityEngine;

public class GhostScatter : GhostBehavior
{
    private void OnDisable()
    {
        ghost.chase.Enable();
    }

    private void OnTriggerEnter2D(Collider2D other)
    {
        Node node = other.GetComponent<Node>();

        if (node != null && enabled && !ghost.frightened.enabled)
        {
            int index = Random.Range(0, node.availableDirections.Count);
            if (node.availableDirections.Count > 1 && node.availableDirections[index] == -ghost.movement.direction)
            {
                index++;
                if (index >= node.availableDirections.Count) {
                    index = 0;
                }
            }
            ghost.movement.SetDirection(node.availableDirections[index]);
        }
    }
}
```

### 📌 **Movement System**
Handles character movement with direction switching and collision detection.

```csharp
using UnityEngine;

[RequireComponent(typeof(Rigidbody2D))]
public class Movement : MonoBehaviour
{
    public float speed = 8f;
    public float speedMultiplier = 1f;
    public Vector2 initialDirection;
    public LayerMask obstacleLayer;

    public new Rigidbody2D rigidbody { get; private set; }
    public Vector2 direction { get; private set; }
    public Vector2 nextDirection { get; private set; }
    public Vector3 startingPosition { get; private set; }

    private void Awake()
    {
        rigidbody = GetComponent<Rigidbody2D>();
        startingPosition = transform.position;
    }

    private void Start()
    {
        ResetState();
    }

    public void ResetState()
    {
        speedMultiplier = 1f;
        direction = initialDirection;
        nextDirection = Vector2.zero;
        transform.position = startingPosition;
        rigidbody.isKinematic = false;
        enabled = true;
    }

    private void Update()
    {
        if (nextDirection != Vector2.zero) {
            SetDirection(nextDirection);
        }
    }

    private void FixedUpdate()
    {
        Vector2 position = rigidbody.position;
        Vector2 translation = direction * speed * speedMultiplier * Time.fixedDeltaTime;
        rigidbody.MovePosition(position + translation);
    }
}
```

### 📌 **Node System**
Defines available movement directions at each node.

```csharp
using System.Collections.Generic;
using UnityEngine;

public class Node : MonoBehaviour
{
    public LayerMask obstacleLayer;
    public List<Vector2> availableDirections { get; private set; }

    private void Start()
    {
        availableDirections = new List<Vector2>();
        CheckAvailableDirection(Vector2.up);
        CheckAvailableDirection(Vector2.down);
        CheckAvailableDirection(Vector2.left);
        CheckAvailableDirection(Vector2.right);
    }

    private void CheckAvailableDirection(Vector2 direction)
    {
        RaycastHit2D hit = Physics2D.BoxCast(transform.position, Vector2.one * 0.5f, 0f, direction, 1f, obstacleLayer);
        if (hit.collider == null) {
            availableDirections.Add(direction);
        }
    }
}
```

### 📌 **Pacman Controller**
Handles Pacman's movement and death sequence.

```csharp
using UnityEngine;

[RequireComponent(typeof(Movement))]
public class Pacman : MonoBehaviour
{
    public AnimatedSprite deathSequence;
    public SpriteRenderer spriteRenderer { get; private set; }
    public new Collider2D collider { get; private set; }
    public Movement movement { get; private set; }

    private void Awake()
    {
        spriteRenderer = GetComponent<SpriteRenderer>();
        collider = GetComponent<Collider2D>();
        movement = GetComponent<Movement>();
    }
}
```

### 📌 **Teleportation (Passage System)**
Allows Pacman to teleport across the map.

```csharp
using UnityEngine;

[RequireComponent(typeof(Collider2D))]
public class Passage : MonoBehaviour
{
    public Transform connection;

    private void OnTriggerEnter2D(Collider2D other)
    {
        Vector3 position = connection.position;
        position.z = other.transform.position.z;
        other.transform.position = position;
    }
}
```

### 📌 **Pellets**
Basic pellets for points and power-ups.

```csharp
using UnityEngine;

[RequireComponent(typeof(Collider2D))]
public class Pellet : MonoBehaviour
{
    public int points = 10;

    protected virtual void Eat()
    {
        FindObjectOfType<GameManager>().PelletEaten(this);
    }

    private void OnTriggerEnter2D(Collider2D other)
    {
        if (other.gameObject.layer == LayerMask.NameToLayer("Pacman")) {
            Eat();
        }
    }
}
```

---

## 🎯 Conclusion
The **Pacman Unity Clone** provides a fun and interactive experience, featuring classic mechanics such as pellet collection, ghost AI, power-ups, and teleportation. This project showcases fundamental game development principles using Unity, including movement systems, AI behavior, and collision handling. 🎮👾
