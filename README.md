# pg.go - A pure Go Postgres driver

## Connecting

		cn, err := pq.Dial("localhost:5432", pq.Values{"user": "blake"}, nil)
		if err != nil {
			log.Print(err)
		}


## Unnamed Prepeared Query

		rows, err := cn.Query("SELECT length($1) AS foo", "hello")
		if err != nil {
			log.Print(err)
		}

		var length int
		for rows.Next() {
			err := rows.Scan(&length)
			if rows.Error() != nil {
				log.Print(rows.Error())
				break
			}

			if err != nil {
				log.Print(err)
				break
			}

			log.Printf("length = %d", length)
		}


## Notifications

**Example**

		// Concurrently read notifications to avoid blocking the connection (see To Know).
		go func() {
			for n := range cn.Notifies {
				log.Printf("notify: %q:%q", n.Channel, n.Payload)
			}
		}()

		cn.Exec("LISTEN user_added")
		cn.Exec("INSERT INTO user (first, last) VALUES ($1, $2)", "Blake", "Mizerany")
		cn.Exec("SELECT pg_notify(user_added, $1 || " " || $2)", "Blake", "Mizerany")

**To Know**

When one or more LISTEN's are active, it is the responsiblity of the user to
drain the `cn.Notifies` channel; Failing to do so causes reads and writes on
the connection to block if there are pending notifications on the connection.