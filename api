export default async function handler(req, res) {
  if (req.method !== "POST") {
    return res.status(405).json({ error: "Method not allowed" })
  }

  try {
    const { username, paket } = req.body
    if (!username || !paket) {
      return res.status(400).json({ error: "Data tidak lengkap" })
    }

    // ===== ENV VARIABLES =====
    const DOMAIN = process.env.PANEL_DOMAIN
    const API_KEY = process.env.PTERO_APP_KEY
    const NEST_ID = process.env.NEST_ID
    const EGG_ID = process.env.EGG_ID
    const LOCATION_ID = process.env.LOCATION_ID

    // ===== RESOURCE MAP =====
    const map = {
      "1gb":[1000,1000,40],
      "2gb":[2000,1000,60],
      "3gb":[3000,2000,80],
      "4gb":[4000,2000,100],
      "5gb":[5000,3000,120],
      "6gb":[6000,3000,140],
      "7gb":[7000,4000,160],
      "8gb":[8000,4000,180],
      "9gb":[9000,5000,200],
      "10gb":[10000,5000,220],
      "unli":[0,0,0]
    }

    if (!map[paket]) {
      return res.status(400).json({ error: "Paket tidak valid" })
    }

    const [ram, disk, cpu] = map[paket]
    const password = username + Math.random().toString(36).substring(2,8)

    // ===== CREATE USER =====
    const userRes = await fetch(`${DOMAIN}/api/application/users`, {
      method: "POST",
      headers: {
        "Authorization": `Bearer ${API_KEY}`,
        "Content-Type": "application/json",
        "Accept": "application/json"
      },
      body: JSON.stringify({
        email: `${username}@panel.local`,
        username,
        first_name: username,
        last_name: "Panel",
        language: "en",
        password
      })
    })

    const userData = await userRes.json()
    if (userData.errors) {
      return res.status(400).json(userData)
    }

    const userId = userData.attributes.id

    // ===== GET STARTUP CMD =====
    const eggRes = await fetch(
      `${DOMAIN}/api/application/nests/${NEST_ID}/eggs/${EGG_ID}`,
      {
        headers: {
          "Authorization": `Bearer ${API_KEY}`,
          "Accept": "application/json"
        }
      }
    )
    const eggData = await eggRes.json()
    const startup = eggData.attributes.startup

    // ===== CREATE SERVER =====
    const serverRes = await fetch(`${DOMAIN}/api/application/servers`, {
      method: "POST",
      headers: {
        "Authorization": `Bearer ${API_KEY}`,
        "Content-Type": "application/json",
        "Accept": "application/json"
      },
      body: JSON.stringify({
        name: `${username}-server`,
        user: userId,
        egg: Number(EGG_ID),
        docker_image: "ghcr.io/parkervcp/yolks:nodejs_18",
        startup,
        environment: {
          INST: "npm",
          USER_UPLOAD: "0",
          AUTO_UPDATE: "0",
          CMD_RUN: "npm start"
        },
        limits: {
          memory: ram,
          swap: 0,
          disk,
          io: 500,
          cpu
        },
        feature_limits: {
          databases: 5,
          backups: 5,
          allocations: 1
        },
        deploy: {
          locations: [Number(LOCATION_ID)],
          dedicated_ip: false,
          port_range: []
        }
      })
    })

    const serverData = await serverRes.json()
    if (serverData.errors) {
      return res.status(400).json(serverData)
    }

    return res.json({
      success: true,
      username,
      password,
      user_id: userId,
      server_id: serverData.attributes.id,
      ram,
      disk,
      cpu,
      panel: DOMAIN
    })

  } catch (err) {
    return res.status(500).json({ error: err.message })
  }
}
