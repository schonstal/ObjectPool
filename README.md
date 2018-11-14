ObjectPool
==========

Instead of creating and destroying new objects all the time, this script reduces garbage by pooling instances, allowing you to seemingly create hundreds of new objects while only actually using a recycled few.

Usage
=====

Normally, when you instantiate and destroy instances of prefabs, you are constantly creating new objects and destroying them at runtime, which can cause runtime garbage collection and occasional framerate drops. ObjectPool can prevent this by pre-instantiating objects for you, which are then re-used instead of being destroyed!

## Spawning Pooled Objects
For example, if I have a Turret that shoots Bullet objects, I can just create 10 bullets and re-use those same 10 objects. The bullets will never be destroyed, just de-activated and re-activated when you respawn them.

To do this with ObjectPool, you just have to call CreatePool() on the prefab that you want to be pooled.
```
public class Turret : MonoBehaviour
{
	public Bullet bulletPrefab;

	void Start()
	{
		//Create a pool with 10 pre-instantiated bullets in it
		bulletPrefab.CreatePool(10);

		//Or you could also pre-instantiate none, and the system will instantiate them as it needs them
		bulletPrefab.CreatePool();
	}
}
```
Now all you have to do is replace all your calls to `Instantiate()` and `Destroy()` with calls to ObjectPool’s `Spawn()` and `Recycle()`. So for example, when the `Turret` shoots, I spawn a `Bullet` instance.
```
public class Turret : MonoBehaviour
{
	public Bullet bulletPrefab;

	public void ShootBullet()
	{
		//Spawn a bullet at my position with my rotation
		bulletPrefab.Spawn(transform.position, transform.rotation);
	}
}
```
When you want to recycle the instance, you can just call `Recycle()` on the component or game object you want to de-spawn. Here we will `Recycle()` our `Bullet` instance when it collides with something.
```
public class Bullet : MonoBehaviour
{
	void OnCollisionEnter(Collider other)
	{
		//De-activate the object and return it to the spawn pool
		gameObject.Recycle();

		//You can also use this:
		//this.Recycle();
	}
}
```
The `Spawn()` function returns a reference to the created instance, so if you want to store it or call any additional methods on it, you can do so. Unlike Unity’s `Instantiate()`, you do not have to cast the return value to a `GameObject` or `Component`.

## Be Careful With Recycled Objects!
Now that your objects are being recycled and re-used, you have to be careful, because if your instances have any variables that change, you’ll have to manually reset them every time a new instance gets spawned. You can do this by using Unity’s `OnEnable()` and `OnDisable()` functions, which will be called whenever your instance is spawned or recycled.

For example, this is incorrect:
```
public class Bullet : MonoBehaviour
{
	public float travelDuration;
	float timer = 0; //Only gets set to zero once!

	void Update()
	{
		timer += Time.deltaTime;

		if (timer >= travelDuration)
		{
			gameObject.Recycle();
		}
	}
}
```
Why? Because our timer variable counts up, but never returns to zero! So the second time this `Bullet` spawns, it will immediately recycle itself. We can easily fix this:
```
public class Bullet : MonoBehaviour
{
	public float travelDuration;
	float timer;

	void OnEnable()
	{
		//Correct! Now timer resets every single time:
		timer = 0;
	}

	void Update()
	{
		timer += Time.deltaTime;

		if (timer >= travelDuration)
		{
			gameObject.Recycle();
		}
	}
}
```
That’s better, now our `Bullet` will correctly reset his timer variable every time it respawns. It’s a little bit annoying having to do this yourself, but the gained speed from using pooled objects is worth it!

