# Running our application on Kubernetes

- We can now deploy our code (as well as a redis instance)

.exercise[

- Deploy `redis`:
  ```bash
  kubectl create deployment redis --image=redis
  ```

- Deploy everything else:
  ```bash
    set -u
    for SERVICE in hasher rng webui worker; do
      kubectl create deployment $SERVICE --image=$REGISTRY/$SERVICE:$TAG
    done
  ```

]

---

## Is this working?

- After waiting for the deployment to complete, let's look at the logs!

  (Hint: use `kubectl get deploy -w` to watch deployment events)

.exercise[

<!-- ```hide
kubectl wait deploy/rng --for condition=available
kubectl wait deploy/worker --for condition=available
``` -->

- Look at some logs:
  ```bash
  kubectl logs deploy/rng
  kubectl logs deploy/worker
  ```

]

--

🤔 `rng` is fine ... But not `worker`.

--

💡 Oh right! We forgot to `expose`.

---

## Connecting containers together

- Three deployments need to be reachable by others: `hasher`, `redis`, `rng`

- `worker` doesn't need to be exposed

- `webui` will be dealt with later

.exercise[

- Expose each deployment, specifying the right port:
  ```bash
  kubectl expose deployment redis --port 6379
  kubectl expose deployment rng --port 80
  kubectl expose deployment hasher --port 80
  ```

]

---

## Is this working yet?

- The `worker` has an infinite loop, that retries 10 seconds after an error

.exercise[

- Stream the worker's logs:
  ```bash
  kubectl logs deploy/worker --follow
  ```

  (Give it about 10 seconds to recover)

<!--
```wait units of work done, updating hash counter```
```keys ^C```
-->

]

--

We should now see the `worker`, well, working happily.

---

## Exposing services for external access

- Now we would like to access the Web UI

- We will expose it with a `LoadBalancer`

.exercise[

- Create a `LoadBalancer` service for the Web UI:
  ```bash
  kubectl expose deploy/webui --type=LoadBalancer --port=80
  ```

- Check the port that was allocated:
  ```bash
  kubectl get svc
  ```

]

---

## Accessing the web UI

- We can now connect to `webui`, on the allocated IP address, to view the web UI

.exercise[

- Open the web UI in your browser (http://service-ip-address/)

<!-- ```open http://node1:3xxxx/``` -->

]

--

Yes, this may take a little while to update. *(Narrator: it was DNS.)*

--

*Alright, we're back to where we started, when we were running on a single node!*
