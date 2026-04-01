<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <title>Planning Tracker</title>
    <script src="https://unpkg.com/react@18/umd/react.development.js" crossorigin></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js" crossorigin></script>
    <script src="https://unpkg.com/@supabase/supabase-js@2"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
  </head>
  <body>
    <div id="root"></div>

    <script type="text/babel">
      const { useEffect, useMemo, useState } = React;
      const { createClient } = supabase;

      const db = createClient(
        "https://rqocweiyclwcuuizkgrh.supabase.co",
        "sb_publishable_8E0NMXPsJCeEmwkhDq1Wwg_yeJ4Mj9q"
      );

      function App() {
        const [project, setProject] = useState({});
        const [stages, setStages] = useState([]);
        const [mode, setMode] = useState("view");

        useEffect(() => {
          const params = new URLSearchParams(window.location.search);
          if (params.get("mode") === "edit") {
            setMode("edit");
          }
        }, []);

        useEffect(() => {
          async function loadData() {
            const { data: projectRow, error: projectError } = await db
              .from("projects")
              .select("*")
              .eq("slug", "demo-project")
              .single();

            if (projectError) {
              console.error(projectError);
              return;
            }

            setProject(projectRow || {});

            const { data: stageRows, error: stagesError } = await db
              .from("stages")
              .select("*")
              .eq("project_id", projectRow.id)
              .order("stage_order", { ascending: true });

            if (stagesError) {
              console.error(stagesError);
              return;
            }

            setStages(stageRows || []);
          }

          loadData();
        }, []);

        async function updateStage(id, patch) {
          if (mode !== "edit") return;

          setStages((prev) =>
            prev.map((s) => (s.id === id ? { ...s, ...patch } : s))
          );

          const { error } = await db
            .from("stages")
            .update({
              note: patch.note,
              fee_flag: patch.fee_flag,
            })
            .eq("id", id);

          if (error) console.error(error);
        }

        const progressPct = useMemo(() => {
          if (!stages.length) return 0;
          const complete = stages.filter(
            (s) => s.status === "complete" || s.status === "active"
          ).length;
          return Math.round((complete / stages.length) * 100);
        }, [stages]);

        const feeItems = stages.filter((s) => s.fee_flag);
        const isEditMode = mode === "edit";

        return (
          <div style={{ padding: 20, fontFamily: "Arial", maxWidth: 900, margin: "0 auto" }}>
            <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", gap: 12 }}>
              <div>
                <h1 style={{ marginBottom: 6 }}>{project.title || "Planning Tracker"}</h1>
                <p style={{ marginTop: 0 }}>{project.subtitle}</p>
              </div>
              <div
                style={{
                  padding: "8px 12px",
                  borderRadius: 999,
                  background: isEditMode ? "#dbeafe" : "#f3f4f6",
                  color: isEditMode ? "#1d4ed8" : "#374151",
                  fontWeight: 700,
                  fontSize: 14,
                  whiteSpace: "nowrap"
                }}
              >
                {isEditMode ? "Edit mode" : "View mode"}
              </div>
            </div>

            <p><strong>{progressPct}% complete</strong></p>

            <div style={{ background: "#f8fafc", border: "1px solid #e5e7eb", padding: 12, borderRadius: 8, marginBottom: 20 }}>
              <strong>How to use this</strong>
              <p style={{ marginBottom: 0 }}>
                Client link: normal page URL. Your editing link: add <code>?mode=edit</code> to the end of the URL.
              </p>
            </div>

            <h2>Timeline</h2>

            {stages.length === 0 && <p>Loading or no data...</p>}

            {stages.map((stage) => (
              <div key={stage.id} style={{ border: "1px solid #ccc", borderRadius: 8, padding: 14, marginTop: 12 }}>
                <strong>{stage.title} — {stage.date_text}</strong>
                <p>{stage.detail}</p>

                {isEditMode ? (
                  <>
                    <textarea
                      value={stage.note || ""}
                      onChange={(e) => updateStage(stage.id, { note: e.target.value })}
                      placeholder="Add notes..."
                      style={{ width: "100%", minHeight: 90, marginTop: 10 }}
                    />

                    <label style={{ display: "block", marginTop: 10 }}>
                      <input
                        type="checkbox"
                        checked={stage.fee_flag || false}
                        onChange={(e) => updateStage(stage.id, { fee_flag: e.target.checked })}
                      />
                      <span style={{ marginLeft: 8 }}>Additional fee</span>
                    </label>
                  </>
                ) : (
                  <>
                    {stage.note ? (
                      <div style={{ background: "#f9fafb", padding: 10, borderRadius: 6, marginTop: 10 }}>
                        <strong>Update</strong>
                        <p style={{ marginBottom: 0 }}>{stage.note}</p>
                      </div>
                    ) : null}

                    {stage.fee_flag ? (
                      <div style={{ background: "#fff7ed", color: "#9a3412", padding: 10, borderRadius: 6, marginTop: 10, fontWeight: 600 }}>
                        Additional fee applies at this stage
                      </div>
                    ) : null}
                  </>
                )}
              </div>
            ))}

            <h2 style={{ marginTop: 28 }}>Fee Items</h2>
            {feeItems.length === 0 ? (
              <p>No fees yet</p>
            ) : (
              feeItems.map((f) => (
                <div key={f.id} style={{ background: "#fff3cd", padding: 10, marginTop: 5, borderRadius: 6 }}>
                  <strong>{f.title}</strong>
                  <p style={{ marginBottom: 0 }}>{f.note || "Additional fee flagged for this stage."}</p>
                </div>
              ))
            )}
          </div>
        );
      }

      ReactDOM.createRoot(document.getElementById("root")).render(<App />);
    </script>
  </body>
</html>
