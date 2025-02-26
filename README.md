using UnityEngine;
using Photon.Pun;

public class Bullet : MonoBehaviourPun
{
    public float speed = 20f;
    public int damage = 20;

    void Update()
    {
        transform.Translate(Vector3.forward * speed * Time.deltaTime);
    }

    void OnTriggerEnter(Collider other)
    {
        if (other.CompareTag("Player"))
        {
            other.GetComponent<PlayerHealth>().TakeDamage(damage);
            PhotonNetwork.Destroy(gameObject);
        }
    }
}using UnityEngine;
using Photon.Pun;

public class PlayerController : MonoBehaviourPun
{
    public CharacterController controller;
    public float speed = 5f;
    public float jumpHeight = 2f;
    public Transform firePoint;
    public GameObject bulletPrefab;

    private float gravity = -9.81f;
    private Vector3 velocity;

    void Update()
    {
        if (!photonView.IsMine) return;

        float moveX = Input.GetAxis("Horizontal");
        float moveZ = Input.GetAxis("Vertical");
        Vector3 move = transform.right * moveX + transform.forward * moveZ;
        controller.Move(move * speed * Time.deltaTime);

        if (Input.GetButtonDown("Jump"))
        {
            velocity.y = Mathf.Sqrt(jumpHeight * -2f * gravity);
        }

        velocity.y += gravity * Time.deltaTime;
        controller.Move(velocity * Time.deltaTime);

        if (Input.GetMouseButtonDown(0))
        {
            FireBullet();
        }
    }

    void FireBullet()
    {
        PhotonNetwork.Instantiate(bulletPrefab.name, firePoint.position, firePoint.rotation);
    }
}using UnityEngine;
using Photon.Pun;

public class PlayerHealth : MonoBehaviourPun
{
    public int maxHealth = 100;
    private int currentHealth;

    void Start()
    {
        currentHealth = maxHealth;
    }

    public void TakeDamage(int damage)
    {
        if (!photonView.IsMine) return;

        currentHealth -= damage;
        if (currentHealth <= 0)
        {
            Die();
        }
    }

    void Die()
    {
        PhotonNetwork.Destroy(gameObject);
    }
}using UnityEngine;
using Photon.Pun;
using Photon.Realtime;

public class MultiplayerManager : MonoBehaviourPunCallbacks
{
    void Start()
    {
        PhotonNetwork.ConnectUsingSettings();
    }

    public override void OnConnectedToMaster()
    {
        PhotonNetwork.JoinRandomRoom();
    }

    public override void OnJoinRandomFailed(short returnCode, string message)
    {
        PhotonNetwork.CreateRoom(null, new RoomOptions { MaxPlayers = 10 });
    }

    public override void OnJoinedRoom()
    {
        PhotonNetwork.Instantiate("PlayerPrefab", Vector3.zero, Quaternion.identity);
    }
}using UnityEngine;

public class ShrinkingZone : MonoBehaviour
{
    public float shrinkSpeed = 0.1f;
    public float minSize = 10f;

    void Update()
    {
        if (transform.localScale.x > minSize)
        {
            transform.localScale -= new Vector3(shrinkSpeed, 0, shrinkSpeed) * Time.deltaTime;
        }
    }
}using UnityEngine;
using UnityEngine.UI;

public class UIManager : MonoBehaviour
{
    public Text healthText;
    public Text killCounter;

    private int kills = 0;

    void Update()
    {
        healthText.text = "Health: " + FindObjectOfType<PlayerHealth>().GetHealth().ToString();
    }

    public void AddKill()
    {
        kills++;
        killCounter.text = "Kills: " + kills;
    }
}
