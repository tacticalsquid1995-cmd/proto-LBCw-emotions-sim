# proto-LBCw-emotions-sim
This is a multi layer approach to creating a working synthetic emotions sim, featuring expanded word engine for over 60+ words / emotional trajectory vectors. featuring shadow sim and updated code. this is a fully working model of dynamic artificial emotions expressed as multi value functions via LBC(w). LBC, a foundation for sacrificial calculus 
import random
import copy


class LBCAgent:
    def __init__(self, name, gender_skew="neutral", initial_L=1.0):
        self.name = name
        self.L_pool = initial_L          # Vitality / Battery
        self.v_cum = 0.0                 # Cumulative trajectory
        self.lambda_regret = 0.3         # Regret sensitivity

        # Weight Profiles
        if gender_skew == "masculine":
            self.weights = {'w_L': 0.8, 'w_B': 1.5, 'w_C': 0.7}
        elif gender_skew == "feminine":
            self.weights = {'w_L': 1.5, 'w_B': 1.0, 'w_C': 0.5}
        else:
            self.weights = {'w_L': 1.0, 'w_B': 1.0, 'w_C': 1.0}

    # -------------------------
    # Core Processing
    # -------------------------

    def _compute_value(self, token_data):
        # Burnout: cost feels heavier when vitality low
        eff_w_C = self.weights['w_C'] * (2.0 if self.L_pool < 0.2 else 1.0)

        return (
            (self.weights['w_L'] * token_data['L']) +
            (self.weights['w_B'] * token_data['B']) -
            (eff_w_C * token_data['C'])
        )

    def process_token(self, token_data, counterfactual_v=0.0):
        v_actual = self._compute_value(token_data)

        # Regret only if counterfactual was better
        regret = self.lambda_regret * max(0.0, counterfactual_v - v_actual)
        v_final = v_actual - regret

        # Metabolic feedback
        self.L_pool += (v_final * 0.15)

        # Emotional decay (prevents runaway charge)
        self.L_pool *= 0.99

        # Clamp vitality
        self.L_pool = max(-2.0, min(2.0, self.L_pool))

        self.v_cum += v_final
        return v_final

    # -------------------------
    # State Reporting
    # -------------------------

    def status(self):
        if self.weights['w_L'] == self.weights['w_B'] == self.weights['w_C']:
            trap = "666 PROFILE"
        elif abs(self.v_cum) < 0.1:
            trap = "STASIS"
        else:
            trap = "ACTIVE"

        return {
            "name": self.name,
            "v_cum": round(self.v_cum, 3),
            "L_pool": round(self.L_pool, 3),
            "state": trap
        }


class WordEngine:
    def __init__(self):
        self.dictionary = {
            "ambition": {"L": 0.2, "B": 1.5, "C": 0.8},
            "sacrifice": {"L": -0.5, "B": 1.2, "C": 1.8},
            "nurture": {"L": 1.2, "B": 0.5, "C": 0.3},
            "fear": {"L": -1.0, "B": -0.5, "C": 1.5},
            "sovereignty": {"L": 1.0, "B": 1.0, "C": -1.0},
            "exempt": {"L": 1.0, "B": 1.0, "C": -2.0},
            "liability": {"L": -1.0, "B": -1.0, "C": 1.0},
            "audit": {"L": -1.0, "B": 0.0, "C": 1.0},
            "consent": {"L": 1.0, "B": 1.0, "C": 0.0},
            "mandate": {"L": 1.0, "B": 0.0, "C": 1.0},
            "sanction": {"L": -2.0, "B": -1.0, "C": 1.0},
            "leverage": {"L": 0.0, "B": 2.0, "C": 1.0},
            "liquidity": {"L": 0.0, "B": 2.0, "C": -1.0},
            "insolvent": {"L": -2.0, "B": -2.0, "C": 2.0},
            "equity": {"L": 1.0, "B": 1.0, "C": 0.0},
            "indemnity": {"L": 0.0, "B": 1.0, "C": -2.0},
            "default": {"L": -2.0, "B": -1.0, "C": -1.0},
            "protocol": {"L": -1.0, "B": 0.0, "C": -1.0},
            "redacted": {"L": -1.0, "B": 0.0, "C": -1.0},
            "coercion": {"L": -1.5, "B": 1.5, "C": 1.5},
            "blackmail": {"L": -2.0, "B": 2.0, "C": 1.0},
            "whistleblower": {"L": 1.5, "B": -1.0, "C": 2.0},
            "narrative": {"L": 0.5, "B": 1.0, "C": -0.5},
            "gaslight": {"L": -1.0, "B": 0.5, "C": -0.5},
            "virtue signal": {"L": 1.0, "B": -0.5, "C": 0.5},
            "rug pull": {"L": -1.5, "B": 2.5, "C": 0.5},
            "fomo": {"L": 0.0, "B": 1.5, "C": 0.5},
            "cancel": {"L": -2.0, "B": -1.5, "C": 1.0},
            "souveraineté (french)": {"L": 1.0, "B": 1.0, "C": -1.0},
            "souveraineté nationale (french)": {"L": 1.2, "B": 0.8, "C": -0.8},
            "souveränität (german)": {"L": 1.0, "B": 1.0, "C": -1.0},
            "soberanía (spanish)": {"L": 1.0, "B": 1.0, "C": -1.0},
            "سيادة (siyada) (arabic)": {"L": 1.0, "B": 1.0, "C": -1.2},
            "主権 (shuken) (japanese)": {"L": 0.8, "B": 1.2, "C": -0.8},
            "免責 (men-seki) (japanese)": {"L": 1.0, "B": 1.0, "C": -2.0},
            "indemnité (french)": {"L": 0.0, "B": 1.0, "C": -2.0},
            "indemnización (spanish)": {"L": 0.0, "B": 1.0, "C": -2.0},
            "责任 (zérèn) (chinese)": {"L": -1.0, "B": -1.0, "C": 1.0},
            "responsabilidad (spanish)": {"L": -1.0, "B": -1.0, "C": 1.0},
            "haftung (german)": {"L": -1.0, "B": -1.0, "C": 1.2},
            "consentement (french)": {"L": 1.0, "B": 1.0, "C": 0.0},
            "consentimiento (spanish)": {"L": 1.0, "B": 1.0, "C": 0.0},
            "同意 (dōi) (japanese)": {"L": 1.0, "B": 1.0, "C": 0.0},
            "mandat (french)": {"L": 1.0, "B": 0.0, "C": 1.0},
            "mandato (spanish)": {"L": 1.0, "B": 0.0, "C": 1.0},
            "授权 (shòuquán) (chinese)": {"L": 1.0, "B": 0.0, "C": 1.0},
            "sanction (french)": {"L": -2.0, "B": -1.0, "C": 1.0},
            "sanción (spanish)": {"L": -2.0, "B": -1.0, "C": 1.0},
            "عقوبة (uquba) (arabic)": {"L": -2.0, "B": -1.0, "C": 1.5},
            "hegemony": {"L": 1.5, "B": 2.0, "C": -1.5},
            "panopticon": {"L": -1.5, "B": 1.0, "C": -0.5},
            "existential risk": {"L": -2.5, "B": -2.0, "C": 3.0},
            "singularity": {"L": 0.0, "B": 3.0, "C": 2.0},
            "cybernetic": {"L": 0.5, "B": 1.5, "C": 0.5},
            "dialectic": {"L": 0.0, "B": 1.0, "C": 1.0},
            "realpolitik": {"L": -0.5, "B": 2.5, "C": 1.0},
            "necropolitics": {"L": -2.0, "B": 1.5, "C": 2.0},
            "hyperstition": {"L": 0.5, "B": 2.0, "C": -1.0},
            "accelerationism": {"L": -1.0, "B": 3.0, "C": 2.5}
        }

    def get(self, token_key):
        return copy.deepcopy(
            self.dictionary.get(token_key.lower(), {"L": 0.0, "B": 0.0, "C": 0.0})
        )

    def random_token(self):
        return copy.deepcopy(
            random.choice(list(self.dictionary.values()))
        )


class LBCSimulation:
    def __init__(self, agents):
        self.agents = agents
        self.engine = WordEngine()

    def interact(self, sender_idx, receiver_idx, token_key):

        sender = self.agents[sender_idx]
        receiver = self.agents[receiver_idx]

        token = self.engine.get(token_key)

        # --- Asymmetric Impact ---
        sender_token = copy.deepcopy(token)
        receiver_token = copy.deepcopy(token)

        # Sender pays slightly higher cost (expression cost)
        sender_token["C"] *= 1.2

        # Receiver feels legitimacy shift stronger
        receiver_token["L"] *= 1.3

        # --- Counterfactual Simulation ---
        alt_token = self.engine.random_token()
        shadow_v_sender = sender._compute_value(alt_token)
        shadow_v_receiver = receiver._compute_value(alt_token)

        # --- Process ---
        v_s = sender.process_token(sender_token, shadow_v_sender)
        v_r = receiver.process_token(receiver_token, shadow_v_receiver)

        # --- Reporting ---
        print(f"\n--- Event: {sender.name} uses '{token_key}' on {receiver.name} ---")
        print(f"Impact | Sender ΔV: {round(v_s,3)} | Receiver ΔV: {round(v_r,3)}")

        for agent in [sender, receiver]:
            s = agent.status()
            print(
                f"{s['name']} | V_cum: {s['v_cum']} | "
                f"L_pool: {s['L_pool']} | {s['state']}"
            )


# -------------------------
# EXECUTION
# -------------------------

if __name__ == "__main__":

    sim = LBCSimulation([
        LBCAgent("Marcus", "masculine"),
        LBCAgent("Elena", "feminine")
    ])

    sim.interact(0, 1, "ambition")
    sim.interact(1, 0, "nurture")
    sim.interact(0, 1, "fear")
