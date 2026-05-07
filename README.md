import itertools
import time

def solve(n: int):
    if n not in (4, 5):
        print("Only n = 4 or 5 are supported.")
        return

    V = list(range(n))
    ordered_pairs = [(u, v) for u in V for v in V if u != v]
    P = len(ordered_pairs)

    # All 4-vertex subsets
    four_subsets = list(itertools.combinations(V, 4))

    # Precompute all directed 4-cycles (canonical form) on each 4-set
    def directed_4cycles_on_set(U):
        cycles = []
        for perm in itertools.permutations(U, 4):
            # canonical: first vertex is the smallest in the 4-tuple
            if perm[0] != min(perm):
                continue
            a, b, c, d = perm
            cycles.append(((a, b), (b, c), (c, d), (d, a)))
        return cycles

    cycles_by_4set = {U: directed_4cycles_on_set(U) for U in four_subsets}

    def compute_pi(arcs_set):
        pi = 0
        for U in four_subsets:
            for cyc in cycles_by_4set[U]:
                cnt = sum(1 for e in cyc if e in arcs_set)
                if cnt >= 2:
                    pi += 1
        return pi

    def canonical_label(arcs_set):
        best = None
        for perm in itertools.permutations(V):
            relabeled = [(perm[u], perm[v]) for (u, v) in arcs_set]
            relabeled_sorted = sorted(relabeled)
            s = ",".join(f"{a}-{b}" for a, b in relabeled_sorted)
            if best is None or s < best:
                best = s
        return best

    # m ranges according to Table 2
    if n == 4:
        m_values = list(range(2, 7))   # 2,3,4,5,6
    else:  # n == 5
        m_values = list(range(2, 9))   # 2,3,4,5,6,7,8

    results = {}
    total_checked = 0

    for m in m_values:
        t0 = time.time()
        max_pi = -1
        examples = []
        comb_count = 0

        for idxs in itertools.combinations(range(P), m):
            comb_count += 1
            arcs = frozenset(ordered_pairs[i] for i in idxs)
            pi = compute_pi(arcs)

            if pi > max_pi:
                max_pi = pi
                examples = [arcs]
            elif pi == max_pi:
                examples.append(arcs)

        # Merge isomorphic digraphs
        iso_map = {}
        for arcs in examples:
            key = canonical_label(arcs)
            if key not in iso_map:
                iso_map[key] = arcs

        results[m] = {
            "max_pi": max_pi,
            "num_graphs_with_max": len(examples),
            "num_iso_classes": len(iso_map),
            "iso_reps": list(iso_map.values())
        }

        t1 = time.time()
        total_checked += comb_count
        print(f"n = {n}, m = {m:2d}  max π = {max_pi:2d}  "
              f"graphs_with_max = {len(examples):6d}  "
              f"iso_classes = {len(iso_map):3d}  "
              f"(checked {comb_count} combinations in {t1 - t0:.2f}s)")

    print(f"\nTotal combinations checked: {total_checked}")

    print("\n----- Detailed representatives (one per isomorphism class) -----")
    for m in m_values:
        info = results[m]
        print(f"\n=== m = {m}   max π = {info['max_pi']}   "
              f"iso_classes = {info['num_iso_classes']}   "
              f"total_graphs_with_max = {info['num_graphs_with_max']} ===")
        for idx, arcs in enumerate(info["iso_reps"], 1):
            arcs_list = sorted(arcs)
            print(f"\nRepresentative #{idx}:")
            print("Arcs (u -> v):")
            print(", ".join(f"{u}->{v}" for u, v in arcs_list))
            adj = [[0]*n for _ in range(n)]
            for u, v in arcs_list:
                adj[u][v] = 1
            print("Adjacency matrix (row = tail, column = head):")
            for row in adj:
                print("".join(str(x) for x in row))

if __name__ == "__main__":
    try:
        n_input = int(input("Enter n (4 or 5): ").strip())
        solve(n_input)
    except ValueError:
        print("Please enter an integer (4 or 5).")
