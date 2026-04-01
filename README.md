<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <title>Planning Tracker</title>
  </head>
  <body>
    <div id="root"></div>

    <script type="module">
      import React, { useEffect, useMemo, useState } from "https://esm.sh/react";
      import ReactDOM from "https://esm.sh/react-dom/client";
      import { createClient } from "https://esm.sh/@supabase/supabase-js";

      const supabase = createClient(
        "https://rqocweiyclwcuuizkgrh.supabase.co",
        "sb_publishable_8E0NMXPsJCeEmwkhDq1Wwg_yeJ4Mj9q"
      );

      function App() {
        const [project, setProject] = useState({});
        const [stages, setStages] = useState([]);
        const [mode, setMode] = useState("view");

        useEffect(() => {
          const params = new URLSearchParams(window.location.search);
          if (params.get("mode") === "edit") setMode("edit");
        }, []);

        useEffect(() => {
          async function load() {
            const { data: p } = await supabase
              .from("projects")
              .select("*")
              .eq("slug", "demo-project")
              .single();

            if (!p) return;

            setProject(p);

            const { data: s } = await supabase
              .from("stages")
              .select("*")
              .eq("project_id", p.id)
              .order("stage_order");

            setStages(s || []);
          }

          load();
        }, []);

        async function updateStage(id, patch) {
          if (mode !== "edit") return;

          setStages((prev) =>
            prev.map((s) => (s.id === id ? { ...s, ...patch } : s))
          );

          await supabase.from("stages").update(patch).eq("id", id);
        }

        const feeItems = stages.filter((s) => s.fee_flag);

        return (
          <div style={{ padding: 20, fontFamily: "Arial", maxWidth: 900, margin: "0 auto" }}>
            <h1>{project.title}</h1>
            <p>{project.subtitle}</p>

            <p><strong>{stages.length} stages</strong></p>

            <h2>Timeline</h2>

            {stages.map((stage) => (
              <div key={stage.id} style={{ border: "1px solid #ccc", padding: 10, marginTop: 10 }}>
                <strong>{stage.title} — {stage.date_text}</strong>
                <p>{stage.detail}</p>

                {mode === "edit" ? (
                  <>
                    <textarea
                      value={stage.note || ""}
                      onChange={(e) =>
                        updateStage(stage.id, { note: e.target.value })
                      }
                      style={{ width: "100%" }}
                    />
                    <label>
                      <input
                        type="checkbox"
                        checked={stage.fee_flag || false}
                        onChange={(e) =>
                          updateStage(stage.id, { fee_flag: e.target.checked })
                        }
                      />
                      Fee
                    </label>
                  </>
                ) : (
                  <>
                    {stage.note && <p>{stage.note}</p>}
                    {stage.fee_flag && <p>Additional fee</p>}
                  </>
                )}
              </div>
            ))}

            <h2>Fees</h2>
            {feeItems.map((f) => (
              <div key={f.id}>
                <strong>{f.title}</strong>
                <p>{f.note}</p>
              </div>
            ))}
          </div>
        );
      }

      ReactDOM.createRoot(document.getElementById("root")).render(
        React.createElement(App)
      );
    </script>
  </body>
</html>
